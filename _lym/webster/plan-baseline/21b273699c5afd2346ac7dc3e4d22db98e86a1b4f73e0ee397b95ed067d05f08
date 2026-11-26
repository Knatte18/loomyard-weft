# Card 8 — board-run-settings-carry

**Create:**
- `internal/configreg#MoveRunSettings`
- `internal/boardengine#RewriteRunSettings`
- `internal/boardengine#Board.boardCriticalSectionRollingBack`
- `internal/hubreconcile#carryMovedRunSettings`
- `internal/hubreconcile#EnsureFailingRunSettingsCarry`
- `internal/hubreconcile_test#TestEnsure_CarriesMovedRunSettingsOnTheBoard`
- `internal/hubreconcile#TestCarryMovedRunSettings_FailedBoardWriteIsSkippedAndLeavesTheBoard`

**Edit:**
- `internal/boardengine#Board.boardCriticalSection`
- `internal/hubreconcile#walkHub`
- `internal/hubreconcile#walkHooks`
- `internal/hubreconcile#ensure`
- `internal/configreg#TestMoveSettings`
- `internal/hubreconcile_test#TestEnsure_UnreachableBoardRemoteSkipsHubWideConfigAndLeavesStampAbsent`

**Extends:**
- `internal/boardengine/doc.go`
- `internal/hubreconcile/doc.go`
- `internal/configreg/doc.go`

**Uses:**
- `internal/configreg#MoveSettings`
- `internal/hubreconcile#mailOrch`
- `internal/hubreconcile#carryHub`

**Intent:** Each hub-wide walk carries the board entries' `run_settings` to the new paths (D7).

`configreg.MoveRunSettings(settings)` applies `MoveSettings` to a module-keyed object decoded from JSON: it marshals the object to YAML with whole numbers kept as integers, moves, and decodes it back, returning the moved object and the lines; an object with no old key comes back equal and with no line.
`TestMoveSettings` gains a row driving it: `{"loom": {"code_max_bounces": 2}, "drift": {"scope": "full"}}` becomes `{"loom": {"code": {"max_bounces": 2}}, "drift": {"scope": "full"}}`.

`boardengine.RewriteRunSettings(boardDir, rewrite, afterSave)` opens the hub's board from its own board config, as `lym board upsert` does, and runs one board write through `boardCriticalSection` under the board's write lock:
it hands every entry's `run_settings` to `rewrite` and keeps every key the rewrite does not change (`PATTERN-tolerant-records`).
Only when some entry changed does it save and render, and the board's detached sync commits and pushes, so a board with no old key gets no write and no commit;
otherwise the mutation returns `noWrite`.
`boardCriticalSection` restores nothing and commits nothing itself: it saves `board.json`, runs `afterSave`, renders and spawns the detached sync, all under the write lock it releases on return (`internal/boardengine/board.go:103-107`, `:181-199`).
Its body moves into `boardCriticalSectionRollingBack`, which takes a third argument, `rollback`, and runs it under the write lock, before the lock is released, when `afterSave` or the render returns an error;
a rollback error is joined to the error it returns.
`boardCriticalSection` calls it with a nil `rollback`, so no other board write changes and no other call site is edited.
`RewriteRunSettings` writes through `boardCriticalSectionRollingBack`:
its mutation reads `board.json`'s bytes before it changes any entry, it hands the caller's `afterSave` on as is, and its `rollback` writes those bytes back.
So a failure at the hook or at the render leaves `board.json` as it was, and the next walk finds the old keys, writes and renders again;
a render that failed part way leaves the rendered views stale until that walk.
No write the rewrite makes renames or removes an entry, so `followDecisionRecords` has no record to move.
The caller's `afterSave` is nil in production.
A commit or push failure in the detached sync never reaches the walk, so the carry neither sees nor retries it, and the moved `board.json` stays saved in the board dir.
It returns the lines of every entry, each prefixed with the entry's slug.

`carryMovedRunSettings` runs in `walkHub` after `reconcileHubWide` has returned with its write committed, outside that write, so the two board writes never nest; a walk whose hub-wide write was skipped runs no carry, and its stale stamp brings the retry.
It calls `RewriteRunSettings` with `MoveRunSettings` and mails the orch one line, `[reconcile notice] lym build <label> moved the board entries' run_settings keys to their new names: <lines>`, through `mailOrch`, when any entry moved.
A failed read or write, any error `RewriteRunSettings` returns, is logged with the error and the way forward `fix the cause the error names; the next verb that loads the loom or tapestry config retries the carry`, and returns skipped, so `walkHub` reports the walk skipped, the stamp stays stale and the next walk retries, as after a skipped hub-wide write.
A failure after the save, at the hook or at the render, leaves `board.json` as it was through the rollback;
only a rollback that fails too leaves the moved entries saved, and the logged error names both failures.
`ensure`'s skipped log names the run_settings carry beside the hub-wide write and the mid-merge worktree, and says the next verb that loads a reconciled config retries.
`walkHooks` gains `afterRunSettingsCarrySave func() error`, which the walk hands to `RewriteRunSettings` as `afterSave`; `EnsureFailingRunSettingsCarry`, in `export_test.go`, runs `ensure` with that hook failing, as `EnsureFailingAfterHubWideWrite` does for the hub-wide write.
Bound: a walk runs once per build per machine, so an old-name entry written after it by an older build waits for the next build's walk; Seed-Child refuses it meanwhile under card 5, naming the new name and `lym board upsert`.

Tests:
- `TestEnsure_CarriesMovedRunSettingsOnTheBoard` (tier1, new, in `internal/hubreconcile`'s integration test file), over a `hubforge` hub whose board holds an entry with `run_settings` `{"loom": {"code_max_bounces": 2}, "drift": {"scope": "full"}}`:
  a stale-stamp `Ensure` leaves the entry holding `{"loom": {"code": {"max_bounces": 2}}, "drift": {"scope": "full"}}` and mails one notice naming the entry and both full names, and a second walk on a stale stamp writes no board commit;
  `EnsureFailingRunSettingsCarry` fails the write after the rewrite is saved, and the entry is left as it was, through `RewriteRunSettings`' rollback, with the build stamp stale; a following `Ensure` carries it.
- `TestCarryMovedRunSettings_FailedBoardWriteIsSkippedAndLeavesTheBoard` (tier0, new, beside `carry_test.go`'s tests), over `carryHub`'s fixture with a valid board config written beside the board, has two rows.
  In each, `carryMovedRunSettings` returns skipped and leaves `board.json`'s bytes as they were.
  - An unparseable `board.json`, so the skip comes from the board read.
  - A `board.json` whose entry holds `{"loom": {"code_max_bounces": 2}}`, with a directory at one of the board config's rendered output paths, so the render fails after the save and the rollback restores the bytes.
- `TestEnsure_UnreachableBoardRemoteSkipsHubWideConfigAndLeavesStampAbsent` (tier1): its board entry holding a flat loom `run_settings` key keeps it, since a skipped hub-wide write runs no carry.

`doc.go` of each package gains a sentence naming its new functions.

**ImpactSummary:** Adds one board write path, one configreg wrapper and one walk step after the hub-wide commit, and splits `boardCriticalSection`'s body into a variant that rolls a failed save back; the walk's stamp now also waits on the carry, and no other board write changes.

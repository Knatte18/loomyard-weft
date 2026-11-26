# Card 8 — board-run-settings-carry

**Create:**
- `plan:internal/configreg#MoveRunSettings` -> `func MoveRunSettings(settings map[string]any) (map[string]any, []string, error)`
- `plan:internal/boardengine#RewriteRunSettings` -> `func RewriteRunSettings(boardDir string, rewrite func(slug string, settings map[string]any) (map[string]any, []string, error), beforeCommit func() error) ([]string, error)`
- `plan:internal/hubreconcile#carryMovedRunSettings` -> `func carryMovedRunSettings(boardDir, label string, hooks walkHooks) (skipped bool)`
- `plan:internal/hubreconcile#EnsureFailingRunSettingsCarry` -> `func EnsureFailingRunSettingsCarry(geom Geometry, opts Options) error`
- `plan:internal/hubreconcile_test#TestEnsure_CarriesMovedRunSettingsOnTheBoard` -> `func TestEnsure_CarriesMovedRunSettingsOnTheBoard(t *testing.T)`
- `plan:internal/hubreconcile#TestCarryMovedRunSettings_UnreadableBoardIsSkipped` -> `func TestCarryMovedRunSettings_UnreadableBoardIsSkipped(t *testing.T)`

**Edit:**
- `internal/hubreconcile#walkHub`
- `internal/hubreconcile#walkHooks`
- `internal/hubreconcile#ensure`
- `plan:internal/configreg#TestMoveSettings`
- `internal/hubreconcile_test#TestEnsure_UnreachableBoardRemoteSkipsHubWideConfigAndLeavesStampAbsent`

**Extends:**
- `internal/boardengine/doc.go`
- `internal/hubreconcile/doc.go`
- `internal/configreg/doc.go`

**Uses:**
- `plan:internal/configreg#MoveSettings`
- `internal/hubreconcile#mailOrch`
- `internal/hubreconcile#carryHub`

**Intent:** Each hub-wide walk carries the board entries' `run_settings` to the new paths (D7).

`configreg.MoveRunSettings(settings)` applies `MoveSettings` to a module-keyed object decoded from JSON: it marshals the object to YAML with whole numbers kept as integers, moves, and decodes it back, returning the moved object and the lines; an object with no old key comes back equal and with no line.
`TestMoveSettings` gains a row driving it: `{"loom": {"code_max_bounces": 2}, "drift": {"scope": "full"}}` becomes `{"loom": {"code": {"max_bounces": 2}}, "drift": {"scope": "full"}}`.

`boardengine.RewriteRunSettings(boardDir, rewrite, beforeCommit)` opens the hub's board from its own board config, as `lym board upsert` does, and runs one board write through `boardCriticalSection` under the board's write lock:
it hands every entry's `run_settings` to `rewrite`, keeps every key the rewrite does not change (`PATTERN-tolerant-records`), and saves, commits and pushes as the board's write path does only when some entry changed, so a board with no old key gets no commit.
`beforeCommit`, nil in production, runs after the rewrite is saved and before the commit; its error fails the write as a commit failure would, and the board's write path restores the store it saved, so a failed write leaves the board as it was.
It returns the lines of every entry, each prefixed with the entry's slug.

`carryMovedRunSettings` runs in `walkHub` after `reconcileHubWide` has returned with its write committed, outside that write, so the two board writes never nest; a walk whose hub-wide write was skipped runs no carry, and its stale stamp brings the retry.
It calls `RewriteRunSettings` with `MoveRunSettings` and mails the orch one line, `[reconcile notice] lym build <label> moved the board entries' run_settings keys to their new names: <lines>`, through `mailOrch`, when any entry moved.
A failed read or write is logged with the error and the way forward `fix the cause the error names; the next verb that loads the loom or tapestry config retries the carry`, leaves the board as it was, and returns skipped, so `walkHub` reports the walk skipped, the stamp stays stale and the next walk retries, as after a skipped hub-wide write.
`ensure`'s skipped log names the run_settings carry beside the hub-wide write and the mid-merge worktree, and says the next verb that loads a reconciled config retries.
`walkHooks` gains `beforeRunSettingsCarryCommit func() error`, which the walk hands to `RewriteRunSettings` as `beforeCommit`; `EnsureFailingRunSettingsCarry`, in `export_test.go`, runs `ensure` with that hook failing, as `EnsureFailingAfterHubWideWrite` does for the hub-wide write.
Bound: a walk runs once per build per machine, so an old-name entry written after it by an older build waits for the next build's walk; Seed-Child refuses it meanwhile under card 5, naming the new name and `lym board upsert`.

Tests:
- `TestEnsure_CarriesMovedRunSettingsOnTheBoard` (tier1, new, in `internal/hubreconcile`'s integration test file), over a `hubforge` hub whose board holds an entry with `run_settings` `{"loom": {"code_max_bounces": 2}, "drift": {"scope": "full"}}`:
  a stale-stamp `Ensure` leaves the entry holding `{"loom": {"code": {"max_bounces": 2}}, "drift": {"scope": "full"}}` and mails one notice naming the entry and both full names, and a second walk on a stale stamp writes no board commit;
  `EnsureFailingRunSettingsCarry` fails the write after the rewrite, leaves the entry as it was and the build stamp stale, and a following `Ensure` carries it.
- `TestCarryMovedRunSettings_UnreadableBoardIsSkipped` (tier0, new, beside `carry_test.go`'s tests): over `carryHub`'s fixture with a board that does not parse, `carryMovedRunSettings` returns skipped and leaves the board file's bytes as they were.
- `TestEnsure_UnreachableBoardRemoteSkipsHubWideConfigAndLeavesStampAbsent` (tier1): its board entry holding a flat loom `run_settings` key keeps it, since a skipped hub-wide write runs no carry.

`doc.go` of each package gains a sentence naming its new function.

**ImpactSummary:** Adds one board write path, one configreg wrapper and one walk step after the hub-wide commit; the walk's stamp now also waits on the carry, and no other board write changes.

---
verdict: APPROVED
prior:
  - id: F11
    status: resolved
  - id: F12
    status: resolved
  - id: F13
    status: resolved
---

No new finding qualifies under the incremental rule: none of the changed hunks of cards 8 and 9 holds a BLOCKING defect.
The approval does not cover the defects under "Real defects not raised".
They sit in plan text the round-2 fixes did not change, and three of them answer the focus directive with "no".

## Prior findings

- F11: `loomPreStepUnmarked` returns the busy refusal at its run-lock probe, before the seed commit and `reed.Up()`. The full wire loads loom's config before that, so the rewritten test fails without the walk and starts no tmux server.
- F12: `runDobby` is the one production caller of `hubDobbyDeps`, and `selfexec` re-execs with `os.Args`, so a re-exec walks again.
- F13: a failure at the hook or the render now runs the rollback under the write lock. `followDecisionRecords` does not return early, and the sync's error is ignored.

## Focus verdicts

- Card 9 tier1 tests and tmux: the shed test holds. No planned test starts tmux or drives a producer. The loom circling test does not reach its load (defect below).
- Card 9 walk sites: holds. I found no further route to a strict loom or tapestry load among the `LoadConfig`, `LoadTapestryConfig`, `SettingLookup` and `ArmAt` callers. `batten cancel`, `cleanup` and `overview` skip `battenCLI.arm` and reach none of those loads.
- Card 8 failure handling: holds. `board.json` keeps its old bytes after a failed save, hook or render, the stale stamp retries the carry, and a planned test reaches the hook and the render.
- `--unset` with two old keys: no, the removal route fails (defect below).
- Card 8 second-walk assertion: no, it cannot fail (defect below).
- Card 9 circling test: no, the verb loads nothing (defect below).

## Real defects not raised

Each was confirmed against the code. None is a finding, because an incremental round raises only on changed hunks.

- **Card 9, line 75: `TestLoomVerbs_StaleStampReconcilesBeforeACirclingVerbLoads` cannot be written as described.** This one would be BLOCKING in a full round, and card 9's own gate will meet it.
  - Bare `lym loom circling` is a group that prints help and loads nothing. It also refuses nothing outside a git repository.
  - `cobra.EnableTraverseRunHooks` is set only in `cmd/lym/main.go`. In a loomcli test, a circling verb runs the group's own pre-run and never the loom pre-run that holds the walk.
  - Only `circling continue`, on a run awaiting at a Bouncer row, loads config. It reads the hard cap, which does not depend on `code_max_bounces`, and prints no soft cap.
  - A working test either drives a verb whose nearest pre-run is loom's own and that loads the seeded key, or sets that cobra global without `t.Parallel` and drives `circling continue`.
- **Cards 2 and 5: D4's `--unset` removal route fails for a file holding two old keys.** This also warrants BLOCKING in a full round.
  - `setChecked` (`internal/configcli/configcli.go`) rolls a write back when the load error after it differs from the one before. Card 2 names the first old key in sorted order, so unsetting that key changes the error to name the next one.
  - At the hub default layer, `setHubWide` runs `validateHubFile` and rolls back while any old key remains.
  - Unsetting every old key in one command works, but each refusal names one key at a time.
  - Card 5's `TestDispatchSet` row holds one old key, so it passes.
- **Card 8, line 63: "a second walk writes no board commit" cannot fail.** `boardCriticalSection` spawns the sync through `selfexec.LymBinary`, which returns `ErrTestBinary` under `go test`, so no carry is ever committed there. A tier0 row would check it: a board with no old key and a directory at a rendered output path must not be reported skipped, since any save and render would fail.
- **Card 9, line 79: the `CodeReviewCaps` row needs a hard cap beside it.** A hub base holding only `code_max_bounces: 7` is refused after the move, because the template's `review.hard_max_bounces` of 4 is below it.
- **Card 6: a walk that fails after the hub-wide commit can lose the move notice.** `walkHub` returns early on a pair's error, and the next walk finds no old key. The card says the lines are mailed "once the walk ends" and does not say a failing walk mails them.
- **Card 11 and `internal/shellguard`: a possible clash between two PATTERN entries.**
  - `pythonRefusal` names `deny_python` bare. Card 11 lists no `shellguard` target, though its search instruction would pull the message in.
  - `PATTERN-config-key-names` asks for `configengine`'s helper, and `PATTERN-leaf-packages` does not let `shellguard` import `configengine`.
  - The plan declares no exception. The parent should settle which entry gives way.

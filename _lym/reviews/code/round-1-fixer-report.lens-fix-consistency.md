Findings (lens fix-consistency):

1. LOW — `internal/shedverbs/doc.go:244-248` (dead-loop stop description). The changed `deadLoopStop` now waits a bounded time for the keeper's exit mark and names the status file's producer only when it differs from the in-flight one. The package doc's "How the loop ended is read from the loop log's lines after the loop's last starting mark" and "With no marks the clause reads 'loop exit: not recorded'" do not mention the wait. A reader of the package doc would still expect an immediate read. The doc.go edit at line 240 covers only the producer clause.

2. LOW — `internal/shedverbs/loop_unix_integration_test.go:3`. The file header still reads "holds the loop test that kills the child tree through its process group, which Windows replaces with a named job." The fix added `TestLoopIntegration_KeeperWaitsThroughATerminatingSignal`, the `openLoopLog` helper and keeper start-failure rows to this file. Those cover keeper behaviour, so the header no longer describes the file. The original `ProcessExitsAreMarked` already sat outside the header, so this is partly pre-existing; the fix widened it.

3. LOW — `internal/shedverbs/loopwait.go:585`. The `loopExitedEarlyStop` comment says "how the loop ended, from the loop log" and it calls `loopExitEvidence` with no wait, while `deadLoopStop` now waits. This twin is consistent only because the waiter spawned the keeper, so a closed spawn channel means the mark was already written. Neither comment states that. This is not a defect, only an unstated reason for the asymmetry.

Checked and consistent:
- `loop.go:88-91` (`OnStart`) and `doc.go:219/228` now agree. Both say the keeper runs it whether or not the loop then refuses.
- The `LoopSpec.Spawn` comment matches `spawnLoop`.
- `status_moved` assertions and rows match `loopExitStop`'s `hasChild() && !inFlightAt(after)`.
- The `ArmingRefusalStopsAsBootstrap` keeper row matches `doc.go:136-140` and `ReportLoopArmError`: the refusal goes to the envelope file, not stdout.

Tool calls made:
1. Grep for twin statements of the changed text across `internal` and `docs`, plus `doc.go:130-145`. I ran it to find `doc.go` and comment twins of the edited text; my context held only the `doc.go` lines the fixer read (215-244).

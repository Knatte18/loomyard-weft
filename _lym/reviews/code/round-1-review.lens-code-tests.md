```yaml
lens: code-tests
outcome: reported
```

Lens: code-tests. For each behaviour the diff adds or changes, the test that pins it, or none.

**Behaviour → pinning test**

1. The waiter spawns the keeper with `--loop-keeper=<id>`. `TestLoopIntegration_ChildTreeDiesWithLoop` pins it: without a keeper no keeper mark appears and `waitUntil` times out. The `loopwait_test.go` fakes only translate the flag.
2. The keeper marks the loop process's exit status. `TestLoopIntegration_ProcessExitsAreMarked` pins it, row "keeper marks the loop process's exit status". The mark opens its own line after unterminated output, so the `loop log: crashed` suffix fails if that breaks.
3. The keeper marks a signal that killed the loop. Pinned by row "keeper marks the signal that killed the loop process" (`signal: killed`) and by `ChildTreeDiesWithLoop` (`signal: terminated`) after the waiter is dead.
4. The waiter's reap marks `keeper process <pid> exited`. Pinned by row "the waiter's spawn marks its keeper's exit status".
5. The keeper outlives the waiter and writes its mark afterwards. Pinned by `ChildTreeDiesWithLoop`, which kills the waiter, then SIGTERMs the loop, then waits for the keeper's mark.
6. `LoopSpec.OnStart` runs once in the keeper after the loop process starts. Pinned for the count only: the `keep` rows check `starts == 1`. Ordering "after the start" is not asserted.
7. The dead-loop stop treats the pid record's child as in flight after the child moved the status file (detail clause, `trace_file`, `trace_copy`, `stderr_path`, stderr excerpt). Pinned by the row `StatusMovedByTheChild_StepFilesStillNamed`.
8. A recorded child that has exited names no step files and uses the loop log as `trace_file`. Pinned by the row `ChildExited_NoInFlightClause_NoStepFiles_LoopLog`.
9. The `loop.status_moved` flag keeps its meaning. A moved-by-child row exists, but its `status_moved` is not asserted anywhere. The existing tests at `loopwait_test.go:469` and `:720` cover the old cases only. Behaviour is pinned only by the unmoved and no-child cases.

**Findings**

- **MEDIUM, design (coverage gap)**: nothing tests that the keeper survives terminating signals and keeps waiting (`internal/shedverbs/loopkeeper.go:30-46`). The `keep` rows stub `Signals` with a nil channel, and `ChildTreeDiesWithLoop` signals only the loop and the waiter. Deleting the handler would pass every test. The mark guarantee then fails in production on teardown. Evidence: `loop_unix_integration_test.go:169`, `:70`.
- **MEDIUM, design (coverage gap)**: the keeper's flag has no test on the arming-refusal path. `DetachedLoopID`'s keeper branch is used by `shedcli` and `loomcli` `reportArmFailure`, and by the loop-id validation message in `stepCmd`. The only arming test uses `LoopDetachedFlag` (`loop_arming_integration_test.go:150`). A grep of the tests finds `LoopKeeperFlag` only in the two test-helper mappers. Reverting `DetachedLoopID` to the detached flag alone would pass every test, and an arming refusal in the keeper's pre-run would then print instead of reaching the waiter as the loop envelope. Evidence: `step.go:310-319`, `loomcli/cli.go:271`, `shedcli/cli.go:205`.
- **LOW, design (coverage gap)**: no row tests the keeper's start-failure path. That path writes a "could not start: <err>" mark, returns 1 and skips `OnStart` (`loopkeeper.go:50-56`). A grep for `could not start` and `keepLoop` in the tests finds nothing for it.
- **LOW, design (coverage gap)**: `keepLoop`'s return value (0 on a clean exit, 1 otherwise) is never asserted. The `keep` helper discards it (`loop_unix_integration_test.go:172`).
- **LOW, design (coverage gap)**: the new `status_moved` behaviour on the moved-by-child row is not asserted. For that row `StatusMoved` is `hasChild && !inFlightAt(after)`, which is true. No assertion fails if the field flips.
- **LOW, design (coverage gap)**: no row covers the new `deadLoopStop` branch where the child is in flight but `status.CurrentProducer == ""`. The ", and the status file now reads" clause is suppressed there. All table rows with a moved status use `"B"`.
- **LOW, design (coverage gap)**: the loop process starts in its own session via `proc.Detach` (`loopkeeper.go:42`). The only coverage is the loop surviving the waiter's SIGKILL, which does not distinguish a session of its own.

**Test economy**

No redundant tests found. `ProcessExitsAreMarked` folds the old spawn-only test into a table, which matches the one-test-per-behaviour rule. Its three rows are distinct behaviours, so nothing needs the `//testtiming:keep` mark. The redundancy report judged none of the three listed tests, so there is no ground to file on.

**Tool calls**
1. `sed` of `loopwait_test.go` lines 626–660 plus greps for `status_moved`, `keepLoop`, `could not start` and `loop-keeper` in tests: my context ended at line 629 of that file, and I needed to confirm the remaining assertions and the absence of keeper tests.

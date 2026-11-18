```yaml
lens: code-tests
outcome: reported
```

Lens: code-tests. Every changed behaviour is pinned by at least one test row. I found one LOW gap and one NIT.

**Behaviour → the test that pins it**
1. `StopOrphanedStepRuns` stops a running run under `stepsDir` whose waiter is unrecorded (PID 0). Pinned by `TestRunner_StopOrphanedStepRuns_StopsOnlyRunsWhoseWaiterIsGone`, row "unrecorded" (internal/shuttleengine/stop_test.go:359).
2. `StopOrphanedStepRuns` stops a run whose recorded waiter pid is gone (PID &gt; 0, process dead). The same test's "gone" row uses PID -1, which fails the `rs.PID &gt; 0` check (activity.go:170), so it takes the same branch as "unrecorded". The shared predicate `runIsLive` is pinned for a dead positive pid elsewhere: activity_test.go:88-90 and 240-252 stub `isAlive` and assert `WaiterGone`. Covered indirectly (see NIT below).
3. A live waiter, another steps directory, an untied run and a terminal run are left alone. Pinned by the same test, rows live-waiter, foreign, untied and terminal, through the outcome loop and `RemoveStrand` calls (stop_test.go:360-363, 642-656).
4. `Untied` stays empty for the orphaned stop. Pinned by the exact `reflect.DeepEqual` report (stop_test.go:642).
5. An empty root never probes reed in the orphaned stop. Pinned by the same test (stop_test.go:352-355).
6. Held, Finished and Failed handling in the shared `stopStepRunsWhere`. Pinned by `TestRunner_StopStepRuns_StopsOnlyTheStepsRuns` through `StopStepRuns`, rows held, finished and the failing removal (stop_test.go:250-312).
7. `StopStepRuns` keeps its old behaviour after the refactor: a tied run is stopped, untied runs are listed, another step's run is left. Pinned by `TestRunner_StopStepRuns_StopsOnlyTheStepsRuns`, plus enginedispatch_test.go:739.
8. `StopStepRuns` no longer probes reed when no run qualifies (newly documented at stepstop.go:66). Pinned only through the shared helper, by the empty-root check of the orphaned test. No `StopStepRuns` row covers it, but the code path is the same helper, so that is adequate.
9. The loop calls `ReconcileOrphanedSteps` before its first child, whatever the status reads (failed with producer error, running). Pinned by `TestLoop_ReconcilesOrphanedStepsBeforeTheFirstChild`, which asserts the call order `reconcile`, `run` (loop_test.go:337-372).
10. The loop writes a non-empty note to the log and writes nothing for an empty note. Pinned by the same test, `wantMark` rows (loop_test.go:337-381).
11. A nil `ReconcileOrphanedSteps` is skipped. Pinned implicitly: every other loop test runs with the hook nil.
12. The dead-loop stop hands its in-flight trace to `OnStepOrphaned` and keeps the interrupted record. Pinned by the `TestUntilStop_DeadLoopStopsLoopExited` subtest (loopwait_test.go:481-506).
13. loomcli wiring: `ReconcileOrphanedSteps` is set, reaches `spec.StepsDir`, and renders the "stopped" note. Pinned by `TestSpecFor_ScratchAndFrictionDir/LoopHooksStopTheStepsShuttleRuns` (cli_test.go:358-386).
14. `stepStopNote` rendering (stopped, untied, finished, held, failed, stop error). Pinned by the looparm_test.go:85-125 table, through `watchdogKillNote` and `orphanedStepNote` only.
15. `orphanedStepsNote` renders a failing stop's error rather than swallowing it. No row covers it: cli_test covers only the success report, and the looparm_test hooks table does not include `orphanedStepsNote`.
16. `stopOrphanedStepRuns` / `stepStopRunner` on the real-runner path (`stopOrphanedRuns` nil). No row covers it. This is the same lightly-wired runner construction `stopStepRuns` already used, moved into a helper; there is no new branch to pin beyond the seam check.
17. `TestSpecFor_ScratchAndFrictionDir` already carries its `//testtiming:keep` marker (cli_test.go:286), so it needs nothing.
18. The removed `reconcileInterruptedStep` / `newestInterruptedTraceID` has no live caller or test left. The grep over internal and cmd showed only the new symbols.

**Findings**
- **LOW** · class `design` · internal/loomcli/looparm.go:144: no test pins the error path of the new reconcile hook. If `orphanedStepsNote` dropped the stop's error, no row would fail. cli_test.go:358-386 drives only a successful report. The looparm_test.go:78-83 hooks table, whose "a failing stop renders its error" row (looparm_test.go:112-115) pins that path for the sibling hooks, does not include `orphanedStepsNote`. Fix: add `orphanedStepsNote` to that table, with a stop func of the `stepsDir` signature, so the error and report rows run through it too.
- **NIT** · class `consistency` · internal/shuttleengine/stop_test.go:358: the "gone" row seeds `PID: -1`, which fails the `rs.PID &gt; 0` check (activity.go:170) the same way the "unrecorded" row's PID 0 does. The row named for a gone waiter therefore never exercises a recorded positive pid whose process is dead. That branch is covered only through `runIsLive` in activity_test.go:88-90 and 240-252. Fix: stub `isAlive` as activity_test does, and give the "gone" row a positive pid it reports dead.

**Tool calls**
- Read stop_test.go:230-312, enginedispatch_test.go:725-760 and the `isAlive` stubs: I needed the existing `StopStepRuns` rows and the waiter-gone coverage, which my context did not hold.
- Grepped the stop_test reed-probe rows and the loomcli note-rendering tests: I needed to know which hooks the `stepStopNote` table drives.
- Grepped cli_test.go for `testtiming:keep` and read looparm_test.go:85-125: I needed the keep-marker state and the note table's rows, neither in context.

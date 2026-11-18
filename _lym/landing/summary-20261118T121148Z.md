# Reconcile a gone step's shuttle runs from run records at each loop start

A re-step could still end on a shuttle run that an earlier step left reading `running`.
Before this change, `reconcileInterruptedStep` in `internal/shedverbs/loop.go` reconciled leftovers only when two things held:
the status error carried the text an interrupted stop writes, and an interrupted record in the steps directory named the step.
A step that ended in an error, or died without writing an interrupted record, matched neither, so its runs were never stopped.

The loop now reconciles from the run records themselves.

## Changes

- **`shuttleengine.Runner.StopOrphanedStepRuns(stepsDir)`** (`internal/shuttleengine/stepstop.go`) stops each running run whose record names a step under `stepsDir` and whose waiting process is gone, or whose record names none.
  It treats each run as `StopStepRuns` treats a run of its step:
  stopped through the stop verb, listed under `Held` when a live process may hold its session, and listed under `Finished` when its output files all exist.
  A run whose waiter lives is left alone.
  So is a run of another steps directory and a run naming no step.
  Both stops now share `stopStepRunsWhere`, which probes reed only once a run qualifies.
  A stop with nothing to act on therefore no longer reports a reed probe failure.
- **`shedverbs.LoopSpec.ReconcileOrphanedSteps`** replaces `reconcileInterruptedStep` and `newestInterruptedTraceID`.
  Every loop calls it once before its first child, whatever the status file reads, and writes a non-empty note to the loop log.
  The loop holds the loop lock then and no child is running, so any run tied to the run's steps whose waiter is gone is a leftover.
  `OnStepOrphaned` keeps its per-child calls: after a child exits without an envelope, after a panic, and in the dead-loop stop.
- **`internal/loomcli`** wires the hook to `StopOrphanedStepRuns` through `orphanedStepsNote`.
  Rendering moved into `stepStopNote`, which the existing notes share.
  Building the runner moved into `stepStopRunner`, so both stops use the same runner.
  `loomCLI.stopOrphanedRuns` is the test seam beside `stopRuns`.

## Design decisions

- The reconcile runs on every loop start, not only on a failed status.
  A leftover can follow any stop, and the check reads the run-directory root, which is cheap.
  Reed is only probed once a run qualifies.
- A record with no waiter pid counts as gone, the same reading `ReadAgentActivity` gives `WaiterGone`.
  At loop start no step of the run can have a run mid-start, so no live run can match.
- The dead-loop stop still writes the in-flight child's interrupted record.
  The re-step no longer depends on that record, so its comment now states the record's remaining purpose.

## Tests

- `TestRunner_StopOrphanedStepRuns_StopsOnlyRunsWhoseWaiterIsGone`: gone and unrecorded waiters are stopped across different steps.
  A live waiter, another steps directory, an untied run and a terminal run are left alone, and an empty root never probes reed.
- `TestLoop_ReconcilesOrphanedStepsBeforeTheFirstChild` replaces the text-keyed test.
  It checks that the hook runs before the first child on a failed status with a producer error and on a running status, and that an empty note writes no mark.
- The dead-loop subtest in `TestUntilStop_DeadLoopStopsLoopExited` now asserts the `OnStepOrphaned` call and the interrupted record directly.
- The loomcli wiring test checks that `ReconcileOrphanedSteps` reaches the steps directory and renders its note.

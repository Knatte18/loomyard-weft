# Reconcile leftover shuttle runs from run records, not status error text

A shed loop's re-step used to reconcile a leftover shuttle run only when the status error text named an interrupted stop.
A step that died without writing an interrupted record left its run reading running, and the re-step could still end on it.
The loop now reconciles from the run records themselves: a run reading running whose waiting process is gone is stopped before the first child starts.

## What changes

- `shuttleengine`: `Runner.StopOrphanedStepRuns(stepsDir)` stops every running run that names a step under `stepsDir` and whose waiting process is gone or never recorded.
  A run whose waiting process lives is left alone, and so is a run naming no step or a step of another steps directory.
  Held, finished and failed runs are reported the way `StopStepRuns` reports them, and `Untied` stays empty.
- `StopStepRuns` and the new stop share one scan, `stopStepRunsWhere`, which takes a selector.
  Reed is now probed only when a candidate run exists, so a run directory with nothing to stop no longer needs a reachable reed.
- `shedverbs`: `LoopSpec.ReconcileOrphanedSteps` replaces `reconcileInterruptedStep` and `newestInterruptedTraceID`.
  `runLoop` calls it once before the first child, whatever the status file reads, and logs its note.
- `loomcli`: the hook is wired to `Runner.StopOrphanedStepRuns` through `orphanedStepsNote`.
  Runner construction moved into `stepStopRunner`, shared by both stops, and `stepStopNote` renders either report.
- Docs in `shedverbs` and `shuttleengine` describe the new hook and stop.

## Design decisions

- The hook runs unconditionally at loop start.
  No step of the run is in flight then, so every run whose waiting process is gone is a leftover.
- Before stopping a candidate, the stop re-reads its record and skips it when it no longer reads running or no longer matches the selector.
  A process may have attached to the run since the scan, and stopping it would end a live agent.
- The interrupted-record write in `loopExitStop` stays; only its comment changed, since the re-step no longer depends on that record.

## To check by hand

Nothing is outstanding beyond the unit tests added in `internal/shuttleengine/stop_test.go`, `internal/shedverbs` and `internal/loomcli`.
No integration run or regression check was run for this change.

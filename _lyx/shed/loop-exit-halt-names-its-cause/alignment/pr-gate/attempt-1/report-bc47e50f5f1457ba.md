---
verdict: APPROVED
---

The diff of pull request 841 at `938152a7c` against `main` meets the board entry `loop-exit-halt-names-its-cause`.
The decision record has no `Added after Discussion` entries, so the board body is the measure.

## Against the board entry

- Its own anomaly kind for a loop that exits on an interrupt or stalled-session signal: `AnomalyInterrupted` (`interrupted-stop`) in `internal/loomengine/anomaly.go`.
  `ClassifyHalt` picks it from the told `interrupted` flag, which `loomAfterInterrupt` in `internal/loomcli/looparm.go` sets.
  A producer-error `failed` halt still classifies as `producer-hard-failure`.
- The cause in the halt note's reason: the dead-loop stop hands the interrupt hook `loop-exited: ` followed by a detail naming the loop, the in-flight step with its trace id, and whether its step tree was still running and was killed (`internal/shedverbs/loopwait.go`).
- The trace file of the in-flight step: `LoopPIDRecord` gains `trace_id` (`internal/shedverbs/looppid.go`, written in `loop.go`), and the hook gets that step's first trace file in place of the loop log while the status file still reads that step.
- Docs in `internal/loomcli/doc.go` and `internal/shedverbs/doc.go` match.

The diff makes no change outside the entry.
The watchdog and status-read stops also take the new kind because they share the interrupt hook.
That falls within the entry's interrupt and stalled-session wording.

## Open risks carried over

- Darn-Review round 1 finding `dead-loop-stale-child-named-in-flight` (a step that has already finished named as in flight) is fixed in the diff by `LoopPIDRecord.inFlightAt`, with a moved-status test row.
  It names no parent correction.
- The record's own risk, that a panicking step child now classifies as `interrupted-stop`, comes from the entry's own design and names no parent correction.

## Overlap

- Peer `orch-strand-cycle` (row 6; this run is on row 21): no overlap.
  Its decision record leaves the `loop-exited` halt-note part of issue #811 to this task, and it changes reed, the orch watcher and creel, none of which this diff touches.
  The rule would land this run first.
- No parent commits are listed.

No board entry is superseded.
`plan-card-rules` and `gate-runs-once` depend on this task, but this run does not do their work.

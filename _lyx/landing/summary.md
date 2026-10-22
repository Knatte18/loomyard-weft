# Loom halt note for an interrupted stop names its cause and step trace

A `lyx loom run` whose loop process died without an envelope halted as `failed` with anomaly `producer-hard-failure`, the reason `loop-exited` and the loop log as `trace_file`.
None of those told a reader what happened: the producer had not failed, the cause was not named, and the loop log has no record of the exit.
Such a stop now carries its own anomaly kind, a reason that names the loop and the step that was in flight, and that step's trace file.

## What changes

- `internal/loomengine`: new anomaly kind `AnomalyInterrupted`, wire spelling `interrupted-stop`.
  `ClassifyHalt` takes an `interrupted` flag; a `failed` state with the flag set classifies as `interrupted-stop`, without it as `producer-hard-failure` as before.
  The flag has no effect on a blocked state.
- `internal/loomcli`: the interrupt hook sets the flag on every halt note it writes, so the loop child that ended without an envelope, the watchdog kill, the status-read stop and the dead-loop stop all classify as `interrupted-stop`.
- `internal/shedverbs`: `LoopPIDRecord` gains `trace_id`, written with the child record when a child starts.
  The dead-loop stop hands the hook the cause `loop-exited: <detail>`, where the detail names the loop id, the in-flight producer and its trace id, and whether the step tree was still running and was killed.
  The hook gets the first trace file `LoopSpec.TraceFiles` lists for that trace id as `trace_file`.
- Docs: `internal/shedverbs/doc.go` and `internal/loomcli/doc.go` describe the new stop and note.

Closes #801

## Design decisions

- The classifier stays pure: the caller tells it the stop was interrupted, rather than it inferring that from the reason text.
- The recorded child counts as in flight only while the status file still reads its producer and history length.
  A loop that died between steps names no step and keeps the loop log as `trace_file`, since the recorded child had finished.
- The envelope's error text keeps the bare `(loop-exited)` token, because drivers and tests read it by that token; only the hook's cause is extended.
- `LoopPIDRecord` is a tolerant record: a pid file without `trace_id` reads as no trace id and falls back to the loop log, as does a trace id with no trace file.
- The watchdog and status-read stops share the hook and are equally not producer failures, so they take the new kind too.
- A step child that panics also ends without an envelope and now classifies as `interrupted-stop`, though the defect is in lyx; its cause still carries the exit status.

## To check by hand

Nothing ran against a real interrupted loom run.
Interrupt a running loop, for example by killing the loop process, and read the halt note: it should show `anomaly: interrupted-stop`, a `loop-exited: ...` reason naming the step and trace id, and that step's trace file.
The early-exit stop (a loop that exits before it records itself, in `internal/shedverbs/loopwait.go`) now hands the hook a `loop-exited: ` cause with the loop log as trace file, and no test asserts those arguments.

# A loop-exited halt note names its cause and the in-flight step's trace

When a loom run's detached step loop died without an envelope, the halt note gave the bare reason `loop-exited`.
It classified as `producer-hard-failure` and pointed `trace_file` at the loop log, which has no record of the exit (issue #801).
Every interrupted stop now gets its own anomaly kind, and the dead-loop stop's note names what the waiter can see about the death.

## What changes

- `loomengine.ClassifyHalt` takes an `interrupted` flag.
  A `failed` halt with the flag set classifies as the new `AnomalyInterrupted` (`interrupted-stop`).
  A `failed` halt without the flag is still `producer-hard-failure`, and the flag changes no `blocked` kind.
- `loomcli`'s interrupt hook sets the flag on its halt note.
  This covers every stop where a step ended without an envelope: a crashed or killed child, a watchdog kill, the status-read stop and the dead-loop stop.
- `LoopPIDRecord` gains `trace_id`, the in-flight child's trace id, written beside the child record when the child starts.
- The dead-loop stop's `loop.detail` now names the producer that was in flight and its trace id.
  It also says when that step's tree was still running and was killed, which is what a loop killed from outside leaves behind.
  The interrupt hook gets `loop-exited: <detail>` as its cause, so the halt note's reason carries it.
  It also gets the in-flight step's first trace file as `trace_file`, and falls back to the loop log only when the record names no trace id or no file is listed.
- The envelope's error text keeps the bare `(loop-exited)` token that drivers and tests match.
- `internal/shedverbs/doc.go` and `internal/loomcli/doc.go` describe the new pid-record key, the detail and the anomaly kind.

## Design decisions

- One kind for every interrupted stop, not one kind per cause.
  The cause text already says whether it was a watchdog kill, an exit status or a dead loop, and the classifier stays pure on what its caller tells it.
- The waiter does not report the loop process's own exit status or signal.
  Only a waiter that spawned the loop sees that status, and the `LoopSpec.Spawn` seam's channel carries no value.
  A signal handler in the loop could not catch a hard kill.
  The loop log named in the detail still holds whatever the process printed.
- A child that panics also ends without an envelope, so it now reads as `interrupted-stop`.
  Its cause still carries the exit status for the reflection to judge.

## Tests

- `TestClassifyHalt` has rows for the interrupted flag on `failed`, `blocked` and `running` states.
- `TestLoomAfterInterrupt_WritesTheHaltNoteAndNoReflection` checks for `anomaly: interrupted-stop`.
- `TestUntilStop_DeadLoopStopsLoopExited` checks the hook's cause and trace file in three cases: a known trace id with listed files, no trace id, and a trace id with no files.
- `TestLoopIntegration_ChildTreeDiesWithLoop` checks that a real loop writes the in-flight child's trace id into its pid record.

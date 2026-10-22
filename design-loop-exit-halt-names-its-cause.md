# loom halt note for a Merge-In loop exit carries only 'loop-exited' as its reason

`loop-exit-halt-names-its-cause` · Task · bug, loom, shed · running · PR-Align

Imported from [issue #801](https://github.com/Knatte18/loomyard/issues/801).

**What happened**

A `lyx loom run` halted with state `failed`, anomaly `producer-hard-failure`, at the `Merge-In` producer.
The halt note's only explanation was the single line `loop-exited`.
It listed the history and a `trace_file` that pointed at the loop log, not at the failing step.

The step's own trace shows the session was running and the post-merge verify later passed (`landingshed: post-merge verify passed`, then `shed: step done ... outcome=done`).
Around the same time the log has `shuttle: session state disagrees ... loop=running state=idle-stalled cause=interrupt stands=loop`.
The halt looks like an interrupted driver or session, not a producer failure.
A resume of the same run then completed `Merge-In` and the run went on to `Publish`.

**Why it is abnormal**

- `producer-hard-failure` names a failure of the producer, but the producer had no failure; the loop exited from outside it.
- `loop-exited` gives no cause (interrupt, signal, crash, stall), so a reader cannot tell a defect from an operator action without digging through step traces.
- The `trace_file` in the note is the loop log, which has no record of the exit.

**What to do**

- Classify a loop that exits on an interrupt or stalled-session signal as its own anomaly kind, not `producer-hard-failure`.
- Put the cause, and the trace file of the step that was in flight, in the halt note's reason.

Provenance: task `tuned-defaults-template`, from `loom-halt.md`.

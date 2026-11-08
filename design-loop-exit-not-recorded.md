# A loop that ends without an envelope records no exit status, so its halt cannot be explained

`loop-exit-not-recorded` · Note · bug, shed, friction

Imported from [issue #88](https://github.com/Knatte18/loomyard/issues/88).

## What happened

A `lym shed step ... --until-stop` run halted as `failed` with anomaly `interrupted-stop` at step Darn-Write, the third step to run after Preflight and Record-Write had completed.
The halt note reads `loop-exited: loop <id> ended without an envelope` and `loop exit: not recorded`.
The only output is the loop log `.lym/shed/<run>/steps/loop.log`, which holds the envelopes of the earlier steps and nothing about Darn-Write.
The history held five entries (Preflight, Record-Write, Drift-Baseline, Drift-Await, Drift-Expect) while the loop log carried envelopes for only the first two, so the log and the history disagree.

## Why it is abnormal

The loop ended with no envelope and no recorded exit code or signal, so neither the operator nor the reflection agent can tell whether the process crashed, was killed, or exited cleanly without emitting an envelope.
The halt note names `loop.trace_copy` and `loop.stderr_path` as the way forward, but the note's own `trace_file` points at `loop.log` rather than a per-step trace, so the diagnostic trail is thin.

## What to do

Record the loop's exit status (code or signal) when the loop ends without an envelope.
Make the halt note's `trace_file` point at the trace of the interrupted step, and include the `loop.stderr_path` content or location in the note.
Check why the loop log lacks envelopes for the Drift-* steps that the history lists.

Provenance: task slug `darn-writer-anatomy`; notes: loom-halt.md
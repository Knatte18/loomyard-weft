# A loop that ends without an envelope records no exit status, so its halt cannot be explained

`loop-exit-not-recorded` · Task · bug, shed, friction · draft

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

## From issue #89

Imported from [issue #89](https://github.com/Knatte18/loomyard/issues/89).

**What happened**
A darn run reached its last step, Friction-Reflect, after Preflight through Finalize all completed.
The loom halted as `failed` with anomaly `interrupted-stop`: `loop-exited: loop ... ended without an envelope`.
The halt note reports `loop exit: not recorded`, and the status error is `shedverbs: step Friction-Reflect interrupted (loop-exited)`.
The only loop output captured was the earlier steps' JSON envelope lines from `.lym/shed/<run>/steps/loop.log`; nothing explains why the loop ended.

**Why it is abnormal**
The loop produced no envelope and no exit status, so the halt cannot be told apart from a crash, a kill, or a normal end of the reflection session.
A run whose every substantive step is done fails at its final bookkeeping step with no recorded cause.

**What to do**
Record the loop's exit status (or signal) when a loop ends without an envelope, and include it in the halt note.
Check whether a Friction-Reflect session that ends normally, or is closed after filing, can be reported as an interrupted loop exit.
If it can, treat a reflection step ending after its report is written as done rather than `failed`.

Provenance: task slug darn-writer-anatomy; notes: loom-halt-2.md

## From issue #91

Imported from [issue #91](https://github.com/Knatte18/loomyard/issues/91).

A `lym shed step` run halted as failed with anomaly `interrupted-stop`: the loop ended without an envelope and the halt reported `loop exit: not recorded`.
It happened twice at the same producer step (`Darn-Write`), the second time after a `goto` re-step.
The halt message points the operator at `loop.trace_copy`, `loop.stderr_path` and `loop.interrupt_policy`, but the note records no cause, so the loop's exit status and reason are lost.
In the first halt the loop log held only the earlier steps' envelopes, none for the interrupted step; the second halt did name the in-flight trace.

This is abnormal: a loop that dies should record its exit status and a reason, so the halt is diagnosable without reconstructing it from logs.
Record the loop process exit code or signal when the loop ends without an envelope, and include it in the halt note.
Also have the halt note name the in-flight trace in every case, as the second halt does.

Provenance: task slug `slack-presence-auto`; notes `loom-halt.md`, `loom-halt-3.md`.
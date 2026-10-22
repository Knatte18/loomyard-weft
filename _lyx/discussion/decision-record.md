# Decision record — loop-exit-halt-names-its-cause

## Goal

A loom halt note for an interrupted stop, the dead-loop `loop-exited` stop above all, names what happened instead of a bare token: its own anomaly kind, a cause a reader can act on, and the trace file of the step that was in flight.

## Scope

- `internal/loomengine/anomaly.go`: a new anomaly kind for an interrupted stop, chosen by `ClassifyHalt` from a told interrupted flag.
- `internal/loomcli`: the interrupt hook's halt note carries that flag, so it classifies as the new kind and never as `producer-hard-failure`.
- `internal/shedverbs`: the loop's pid record carries the in-flight step's trace id;
  the dead-loop stop hands the interrupt hook a cause naming the loop, the in-flight step and whether its step tree was still alive, and the in-flight step's trace file in place of the loop log.
- Docs: `internal/shedverbs/doc.go` and `internal/loomcli/doc.go` where they describe the loop-exited stop and the interrupted halt note.

Out of scope: the shuttle's session-state disagreement log line, the loop's own signal handling, and the exit status of the loop process.

## Decisions

### An interrupted stop is its own anomaly kind

Decision: add `AnomalyInterrupted` with the wire spelling `interrupted-stop`.
`ClassifyHalt` takes an `interrupted` flag; a `failed` state with the flag set classifies as `interrupted-stop`, without it as `producer-hard-failure` as before.
Every halt note the interrupt hook writes sets the flag: the loop's child that ended without an envelope, the watchdog kill, the status-read stop and the dead-loop stop.

Rationale: every one of those stops is a step that ended without an envelope, from outside the producer, so none is a producer failure.
The flag is told by the caller, keeping the classifier pure.

### The loop-exited cause names the dead loop and its in-flight step

Decision: the cause the dead-loop stop hands the interrupt hook reads `loop-exited: ` followed by the stop's detail, which names the loop id, the in-flight producer and its trace id when the pid record names a child, and whether the step tree was still alive and killed.
The envelope's error text keeps the bare `(loop-exited)` token.

Rationale: the waiter cannot see why the loop process died, but it can tell a loop that died under a live step, which an outside kill leaves, from one whose step was already gone, and it can name the step and its trace.
The envelope's error text is read by drivers and tests by its token, so it stays put.

### The pid record carries the in-flight step's trace id

Decision: `LoopPIDRecord` gains `trace_id`, written with the child record when a child starts.
The dead-loop stop hands the interrupt hook the first trace file `LoopSpec.TraceFiles` lists for that id, and the loop log only when the record names no trace id or no file is found.

Rationale: the loop log has no record of the exit; the step's trace does.
The record is a tolerant JSON record, so an older pid file without the key reads as no trace id.

## Constraints

- `PATTERN-friction-capture`: the halt note stays Go-authored, written through the existing hook, and a note-write failure changes no exit.
- `PATTERN-sole-parsers`: the note is rendered and parsed only through `internal/haltnote`; its format does not change.
- `PATTERN-tolerant-records`: the pid record gains a key and never refuses an unknown one.
- `PATTERN-comment-line-breaks`: one sentence per line in Go comments.

## Auto-mode assumptions

- `interrupted-stop` is the wire spelling; no consumer outside `loomengine` and its tests reads the kind set.
- The watchdog and status-read stops take the new kind too, since they share the interrupt hook and are equally not producer failures.

## Open risks

- A step child that panics ends without an envelope and now classifies as `interrupted-stop`, though the defect is in lyx; its cause still carries the exit status, so the reflection can still tell.

## Acceptance criteria

- A halt note the interrupt hook writes carries `anomaly: interrupted-stop`; a `failed` halt from a producer error still carries `producer-hard-failure`.
- `ClassifyHalt` table covers the interrupted flag on a failed state and its having no effect on a blocked state.
- A dead loop with an in-flight child whose trace file exists hands the interrupt hook a cause starting `loop-exited: ` that names the producer and trace id, and that trace file as the trace file.
- A dead loop whose record names no trace id hands the hook the loop log.
- The envelope's error text keeps the `(loop-exited)` token.

# raddle-propose refuses a note dependency early; a loop exit without an envelope shows its log tail

`raddle-dep-and-loop-exit` · Task · bug, shed, board, loom · running · Darn-Write

First read what landing d1535412 (shedverbs: record how an --until-stop loop's process ended) already covers of the second note; do only what is left, and the first note whole.

## From note raddle-propose-note-deps

## Seen (2026-11-11, raddle of gate-scope-by-coverage-map)

The raddle proposal gave part `testmap-cross-package` a `depends_on` on the note `testmap-upkeep`.
`raddle-propose` validated the proposal and passed it, and the operator was asked to approve it.
Raddle-File then refused the entry because a note cannot be depended on, and the run failed at Raddle-File.
The refusal text ends in "promote \"testmap-upkeep\" with ;", so the command it names is missing.

## Idea

Make `raddle-propose` refuse a `depends_on` that names a note, with the way forward (promote the note, or drop the dependency), so the check runs before the operator approves.
Complete the Raddle-File refusal text with the command it means.

## From note loop-exit-log-tail

Imported from [issue #111](https://github.com/Knatte18/loomyard/issues/111).

## What happened

A `lym loom` run halted as `failed` with anomaly `interrupted-stop` at step `Code-Bouncer`.
The status error was `shedverbs: step Code-Bouncer interrupted (loop-exited)`.
The note reported `loop exit: not recorded`, so neither the exit code nor the signal that ended the loop was available.
The "loop log" field in the halt note held only the JSON envelopes of earlier steps (`Code-Bouncer`, `Code-Burler-Review`, `Code-Burler-Fix`, all outcome `stuck`) rather than anything from the interrupted loop's own end.
The history shows the same three steps already ran with outcome `stuck` before the interrupted `Code-Bouncer` invocation.

## Why it is abnormal

The halt cannot be diagnosed from what lym records.
The way forward tells the operator to read `loop.trace_copy` and `loop.stderr_path` and apply `loop.interrupt_policy`, but the note gives no hint why the loop ended without an envelope (crash, kill, timeout, operator interrupt).
Without a recorded exit status, an intentional interrupt and a lym crash look the same.

## What to do

- Record the loop process's exit code or terminating signal in the halt note and in the run status when a loop ends without an envelope.
- Make the "loop log" field show the tail of the interrupted loop's own output, not the envelopes of steps that completed earlier.
- When the exit cannot be captured, say why in the note instead of `not recorded`.

Provenance: task slug `models-full-buildsection`; note `loom-halt-2.md`.


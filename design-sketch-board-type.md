# A Sketch is a board type of its own

`sketch-board-type` · Task · feature, board, loom · running · Drift-Await

## Idea (operator, 2026-11-05)

An entry whose motif run ended with an approved decision record is a Sketch, and the board shows it as one: a note type of its own beside `research`, `bug`, `enhancement` and `feature`.

- The board gains the note type label `sketch` (handle letter `S`).
- A Sketch is not a task and never runs: it is the stage before the triage command in `discussed-task-split` (F14, "Triage of a Sketch").
  That command breaks the Sketch down and alone decides what it becomes: one or more tasks, each with its own type and recipe, or a project.
- When a motif run's Discussion is approved and the record is stored, lym makes the entry a note of type `sketch`, keeping its module labels and dropping its former type, since the triage gives each resulting task its type.
- The README and `lym board list --text` list Sketches under their own heading.

# A Sketch is a board type of its own

`sketch-board-type` · Note · feature, board, loom

## Idea (operator, 2026-11-05)

An entry whose motif run ended with an approved decision record is a Sketch, and the board shows it as one: a type of its own beside `research`, `bug`, `enhancement` and `feature`.

- The board gains the type label `sketch` (handle letter `S`).
- When a motif run's Discussion is approved and the record is stored, lym gives the entry the type `sketch` in place of its former type label, keeping its module labels; the README and `lym board list --text` list it under its own heading.
- A Sketch is the input of the triage run in `discussed-task-split` (F14, "Triage of a Sketch"), which decides what becomes of it.

## Open

- Whether the entry stays kind `task` (it can then run tapestry from Plan directly) or becomes kind `note` until the triage's proposal is approved.
- Whether the former type label is kept somewhere, since a Sketch may become a mix of bugs and features.

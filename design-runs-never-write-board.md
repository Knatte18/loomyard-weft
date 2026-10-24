# A run never files board entries itself

`runs-never-write-board` · Note · bug, board, loom, stencil

## Seen

`gate-runs-once`'s card 31 filed a new task, `drop-transitional-tag-spellings`, on the board with `lyx board upsert`, as decided in its own decision D13.
It depended on `gate-runs-once`, so the Land notice named it "newly ready" and the orch started it as a full loom run the operator never approved.

## Rule (operator, 2026-10-24)

A run never writes the board itself: no card, decision or producer files a task or note.
A follow-up a run finds goes into its report or decision record, and the orch puts it to the operator, who decides whether it becomes a board entry.

## Idea

Refuse `lyx board` writes from inside a run (the board lock already refuses edits to the run's own entry; extend it to every write),
and tell Discussion and Plan writers in their stencils that follow-ups are reported, never filed.

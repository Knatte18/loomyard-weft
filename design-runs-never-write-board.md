# A run never files a task, at most a note

`runs-never-write-board` · Task · bug, board, loom, stencil · running · Code-Burler-Review

## Seen

`gate-runs-once`'s card 31 filed a new task, `drop-transitional-tag-spellings`, on the board with `lyx board upsert`, as decided in its own decision D13.
It depended on `gate-runs-once`, so the Land notice named it "newly ready" and the orch started it as a full loom run the operator never approved.

## Rule (operator, 2026-10-24)

A run never files a task: only the operator decides what becomes a task, and with which recipe and profile.
A run may at most file a follow-up as a note, which waits on the board like any other note.

## Idea

Refuse a `lyx board` write from inside a run that creates or promotes a task (the board lock already refuses edits to the run's own entry),
and tell Discussion and Plan writers in their stencils that a follow-up becomes a note, never a task.
The orch starts a "newly ready" task only when the operator approved it.

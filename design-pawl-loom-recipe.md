# Pawl as a loom recipe, with a decision record and no review rounds

`pawl-loom-recipe` · Note · enhancement, pawl, loom

## Seen (operator, 2026-10-31)

Pawl takes one board entry, a project, and ends with its beam merged into the parent, as a loom recipe takes a task and lands it.
It has no decision record: its Code-Review perch compares against a snapshot of the project's board entry, and `lym loom decision add` cannot change a beam's scope.

## Idea

Pawl becomes a loom recipe once `run-verbs-shared` has made `loom` the task-run contract, with the members' runs in the writer's place.
It gains a decision record, so `lym loom decision add <beam-slug>` changes a beam's scope as it does a task's.
Its Code-Review rounds are set to zero, since pawl is to have no review round of its own (the note `parent-merge-task`).

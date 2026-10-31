# Pawl becomes a loom recipe, with a project decision record and no review rounds

`pawl-loom-recipe` · Task · enhancement, pawl, loom, board, workspaces · done

## Seen (operator, 2026-10-31)

Pawl takes one board entry, a project, and ends with its beam merged into the parent, as a loom recipe takes a task and lands it.
It has no decision record: its Code-Review perch compares against a snapshot of the project's board entry, and `lym loom decision add` cannot change a beam's scope.

## Design (operator, 2026-10-31)

Pawl becomes a loom recipe under the contract `run-verbs-shared` builds, with the members' runs in the writer's place.
Its Code-Review rounds are zero: pawl has no review round of its own (the note `parent-merge-task` replaces the perch).

Pawl gets a decision record that holds only what binds the whole project: its goal, its bounds and the constraints every member follows.
It is no log of what landed; git and the board hold that, and a member landing never touches it.
- `lym beam create` writes it mechanically: the project entry's body verbatim under `## Goal`, every other section empty.
- After that only a scope change touches it, through `lym loom decision add <beam-slug>`, which the beam's own orch runs.
- `decision add` on a beam tells the live agents of every member run under it that the project record holds a new decision.
- Each member's Discussion and writer read the project record as constraints, and the beam's Describe reads it when the beam lands.

lym enforces the route:
- Once a project has a record, `lym board upsert` refuses a change to its body and names `lym loom decision add <beam-slug>`, as the board already refuses scope edits on a run-held task.
- The orch role lym ships says that a beam's scope change goes through `decision add` on the beam.

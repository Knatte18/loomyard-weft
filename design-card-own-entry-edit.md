# Card can require a board-entry edit that the running task's own board lock refuses

`card-own-entry-edit` · Note · bug, loom, stencil, board

Imported from [issue #653](https://github.com/Knatte18/loomyard/issues/653).

A planned card asked the implementer to cut the task's own board entry body to a pointer through `lyx board upsert`.
The board refuses the edit: the entry is held by the very run executing the card (status `running · Webster`) and takes no scope edits.
Clearing the status is the orch's call and only possible once no run holds the entry, so the implementer could not complete the step and did not work around it.

This is abnormal because the pipeline produced a card whose step is impossible from inside the run it belongs to.
Card planning should assign a "cut the board entry to a pointer" step to the landing or the orch, not to the implementer, or `lyx board upsert` should allow the holding run to edit its own entry's body.

Provenance: task `creel-core`, note `12-hubgeom-creel-geometry.md`.

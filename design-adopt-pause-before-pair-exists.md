# An adopted run cannot be paused before adopt adds its pair

`adopt-pause-before-pair-exists` · Note · bug, tender, fabric

## Seen

On 2026-11-09, before `lym adopt`, `lym tender pause` on the runs owned by osl-1046 failed with `has no pair in this hub`.
The pairs exist only after adopt adds them, so the operator cannot hold a run before taking it over.

## Idea

Adopt takes an option to leave every adopted run paused, or the pause verb records a pause by slug for a run whose pair is not here yet.
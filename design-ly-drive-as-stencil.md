# The ly-drive skill becomes a stencil prompt

`ly-drive-as-stencil` · Note · enhancement, shed, stencil

The `ly:ly-drive` skill exists only to start the drivers that run a loom through Shed with repeated `lyx shed step`.
Nothing about that needs to be a skill: every other Claude process lyx spawns gets its instructions from a stencil prompt, and the driver should too.

- Move ly-drive's content into a stencil under `contracts/stencils/` and have the launch prompt render it.
- Remove the `ly-drive` skill from the `ly` plugin.

`ly:board` stays a skill: it serves the operator's own sessions and the orchestrator, not a spawned process, and it becomes the only skill Loomyard ships.

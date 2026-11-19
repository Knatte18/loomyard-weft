# A Merge danger section in every change description: one-way or two-way door, and blast radius

`merge-danger-in-description` · Note · enhancement, landing, stencil

Source: `docs/research/survey/pocock-skills-2026-11-19/report.md` (survey of Matt Pocock's skills repository, 2026-11-19), candidate 1, which holds the design.

## Idea

The change description gets a `## Merge danger` section: **Door** `one-way` or `two-way` (one-way when a revert does not undo the effect: a format another binary reads, a config reinterpretation, deleted data, anything pushed outward) and **Blast radius** in one word.
The darn and Describe stencils ask for it, `lym loom validate-description` refuses a description without it, and a Code-Review rubric line checks the door against the diff.
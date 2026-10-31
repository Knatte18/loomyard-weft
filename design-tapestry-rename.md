# Rename today's loom recipe to tapestry with a rename program

`tapestry-rename` · Task · enhancement, loom, shed, stencil · draft

Depends on: [`dobby-start-failures`](design-dobby-start-failures.md), [`readme-rewrite`](design-readme-rewrite.md)

## Goal

Today's loom recipe is renamed `tapestry`, mechanically, so that every `loom` left afterwards is a candidate for the general run (`run-verbs-shared`).
`loom` becomes the name of the general run in Shed; what is called loom today is one variant of it, darn another (operator, 2026-10-29).
`tapestry` is a picture woven to a full-size drawing made first: the discuss-and-plan-before-building path.

## Change

- A Go program under `tools/`, like `tools/wordswap` for the `lyx` to `lym` rename (`ef912d41`), carries out the rename, and the run applies it and commits the result.
- The `lym loom ...` verbs keep their names, since they become the general ones.
- No behaviour changes: this is a rename only.

## Open, settled by the run

- Which packages, docs and stencils are the variant's and which are general; the program's inclusion and exclusion lists follow from it.
  `internal/loomcli` already serves darn runs, so it stays `loom`.
- What the board's `recipe` field and an empty `recipe` mean, since existing entries and runs carry `loom` or nothing.
- The strand names such as `ly:LOOM` and the texts that name them.
- Every stencil, way-forward text and doc that says "loom" for the variant changes with it.

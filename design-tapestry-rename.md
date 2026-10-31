# Rename today's loom recipe to tapestry with a rename program

`tapestry-rename` · Task · enhancement, loom, shed, stencil, config, board · running · Code-Burler-Review

## Goal

Today's loom recipe is renamed `tapestry`, mechanically, so that every `loom` left afterwards is a candidate for the general run (`run-verbs-shared`).
`loom` becomes the name of the general run in Shed; what is called loom today is one variant of it, darn another (operator, 2026-10-29).
`tapestry` is a picture woven to a full-size drawing made first: the discuss-and-plan-before-building path.

## Change

- A Go program under `tools/`, like `tools/wordswap` for the `lyx` to `lym` rename (`ef912d41`), carries out the rename, and the run applies it and commits the result.
- The `lym loom ...` verbs keep their names, since they become the general ones.
- No behaviour changes: this is a rename only.

## Bolt as well

The rename covers the records side, Bolt, not only the code: what lym keeps there for the variant is renamed with it, so code and records agree after landing.
That includes the hub-wide config (such as `_lym/config/loom.yaml`), the board copies of stencils and specs (`_lym/stencils/loom`, `_lym/specs`), the orch rules, and the board entries and run records that name the variant, such as a `recipe` of `loom`.
The program renames Bolt the same way it renames the code, and a hub on another machine, or another hub, picks the rename up by a migration lym runs itself, never by hand.

## Open, settled by the run

- Which packages, docs and stencils are the variant's and which are general; the program's inclusion and exclusion lists follow from it.
  `internal/loomcli` already serves darn runs, so it stays `loom`.
- What the board's `recipe` field and an empty `recipe` mean, since existing entries and runs carry `loom` or nothing.
- The strand names such as `ly:LOOM` and the texts that name them.
- Every stencil, way-forward text and doc that says "loom" for the variant changes with it.

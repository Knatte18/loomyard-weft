# A project slug on the board links to its design file

A project with a body renders to `design-<slug>.md`, but nothing on the rendered board linked to it.
Task and note slugs already linked to their design files through `slugCell`;
a project's facts line showed its slug as a plain code span.

## What changes

`boardView.projectFacts` in `internal/boardengine/render.go` now builds the slug part with `slugCell`, the same helper the tables' Slug column uses.
The facts line appears under a project's heading in `README.md`, in any parent project's file, and in the header of the project's own file, so all three places now link the slug to `design-<slug>.md` when the project has a body.
A project without a body keeps the plain code-span slug, since `slugCell` only links when the body is non-empty.

## Docs and tests

- The `boardengine` package doc's README section now says the facts-line slug links to the design file when the project has a body.
- `TestRenderReadmeGolden`'s project row gives the `ship` project a body, so its facts line in `README.md` and in `project-ship.md` expects the link, while the bodyless `later`, `inner`, `tidy` and `orphan` projects pin the plain slug.

## Design decisions

- Reusing `slugCell` keeps one definition of how a slug links to its design file, so the facts line and the tables cannot drift apart.

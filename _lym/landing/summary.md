# Board links a project's slug to its design file

A project's body renders to `design-<slug>.md`, but nothing on the rendered board linked to it, so the body could not be read there.
A project's facts line now links its slug to that file when the project has a body, in `README.md` and in project files alike.
A project without a body keeps the plain code-span slug.

## Design

`projectFacts` in `internal/boardengine/render.go` builds the slug part with `slugCell`, the helper that already links a task's or note's slug in the tables.
A project's link therefore follows the same body check and design prefix as every other slug link, with no second rule to keep in step.
The `internal/boardengine/doc.go` description of the facts line and the matching test comment are updated.

## Tests

The rendering golden for a project file and for `README.md` now gives the project `ship` a body and expects `` [`ship`](design-ship.md) ``.
The sibling project `tidy` has no body, and its facts line keeps the plain slug.

Not run by hand: no rendered board was opened to follow the link in a viewer.

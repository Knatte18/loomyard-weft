# Board README marks a discussed entry and links its Sketch

The board files did not show which entries carry an approved decision record, so a finished Discussion was invisible from the README.

## What changes

- A board table's Slug cell now stacks a `[Sketch](decision-<slug>.md)` link under the slug, with `<br>`, when the entry carries the `discussed` flag.
  The link sits in the cell that already links the design doc, so the row gains no column and no width beyond the word `Sketch`.
- Every table renders its Slug cell through the one new `slugColumnCell` in `internal/boardengine/render.go`, so the mark shows in every table and board file that lists the entry: README.md, the project files, the open-task tables, Notes and Done.
- An entry without the flag renders exactly as before.
- A project's facts line keeps the plain `slugCell`, since a project runs no Discussion and the facts line is a paragraph, not a table cell.
- The link target is the record file `DecisionFileName` names, beside board.json and the rendered files, so the relative link resolves.

## Design decisions

- The mark goes under the slug rather than as a bullet in the title cell or beside the recipe.
  The Recipe column is missing from the Notes and Done tables, and the title cell already stacks the brief and run settings, while the Slug cell is in every table and already holds the link to the entry's design file.

## Docs

The `boardengine` package doc's README section describes the Sketch link in the Slug cell, and its Decision record section says the render links the record file while still never writing or deleting it.

## Tests

`TestRenderSlugCellLinksTheDecisionRecord` in `internal/boardengine/render_test.go` pins a discussed task in the Ready table and a discussed done task in the Done table carrying the Sketch link, and an unflagged task rendering its slug alone, in both README.md and project-root.md.

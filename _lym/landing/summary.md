# Board README links a discussed entry's Sketch under its slug

The board README gave no sign of which entries carry an approved decision record, so the operator could not tell that a `motif` run had finished an entry's Discussion.
A row for an entry flagged `discussed` now shows a `Sketch` link to `decision-<slug>.md` under its slug, in every table of every board file that lists it, Done included.
An entry not flagged discussed renders as before.

## Design decisions

- The link sits in the Slug cell, stacked under the slug with `<br>`, so the mark adds no column and the row stays as narrow as before.
  The slug link to `design-<slug>.md` is unchanged.
- `slugColumnCell` in `internal/boardengine/render.go` wraps `slugCell` and is the only change to row building; `table` calls it for every row.
- The link target is `DecisionFileName(slug)`, relative to the rendered file, like the design-doc link.
  The decision record is still no render output, so the manifest never holds it and a render never deletes it.
- A Done entry keeps its link, since a landed entry keeps the record it last stored.
- `docs/overview.md` and the `internal/boardengine` package docs (`doc.go`, `render.go`) describe the new cell.

## Tests

`TestRenderSlugCellLinksTheDecisionRecord` pins both renderings in `README.md` and `project-root.md`: a discussed open entry, an entry not flagged, and a discussed Done entry.

## To check by hand

- The tests were not run in this step.
  Run `lym gate test ./internal/boardengine`.
- Open a rendered README and confirm the `Sketch` link resolves to `decision-<slug>.md` beside `board.json`.

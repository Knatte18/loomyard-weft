# Board: link a discussed entry's design page to its decision record

A Sketch opened from the board showed only its design page, which holds the entry's original brief.
The decision record, the substance of the Sketch, sat beside it as `decision-<slug>.md` with no link from the design page.
The README tables already linked it from the Slug cell, but the design page did not, so a reader on a phone could not reach it.

## What changes

- The design doc of a discussed entry carries a `Decision record: [Sketch](decision-<slug>.md)` line under its metadata line, near the top.
- This holds for every discussed entry, not only a Sketch, and for a task a Sketch is promoted to.
- `RenderToDisk` renders a discussed entry whose record file is missing from the board directory as undiscussed.
  No board file, table or design page, then links a missing record.
- `Render` itself stays a pure function of the entries and takes the record to exist.
  The file check lives in `RenderToDisk`, which is the caller that knows the board path.
- The `boardengine` package doc and the `Render` and `RenderToDisk` comments describe the link and the missing-file rule.

## Design decisions

- The existence check runs on a copy of the entries, so the caller's entries keep their discussed flag.
  A test pins this.
- The record link uses the same `Sketch` text and `sketchLink` helper as the Slug cell, so the two links cannot diverge.

## Tests

New tests in `render_test.go` and `decision_test.go` cover the design-page line, the same-write link from a stored record, and the missing-file case.
Nothing was left to run by hand.

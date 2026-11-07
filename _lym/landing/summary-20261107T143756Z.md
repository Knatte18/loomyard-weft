# Board: a discussed entry's design doc links its decision record

A Sketch opened from the board showed its design doc, `design-<slug>.md`, which holds only the entry's body.
The decision record beside it, `decision-<slug>.md`, was reachable only through the `Sketch` link in the entry's table row,
and nothing on the design doc pointed at it.

## What changes

- **Design doc link.**
  `renderDesigns` adds a `Decision record: [Sketch](decision-<slug>.md)` line under the metadata line of every entry flagged discussed,
  before the `Depends on:` line.
  This covers a Sketch, any discussed task, a task a Sketch is promoted to and a raddle run's filed parts, since all carry the discussed flag.
  The table rows already link the record through `slugColumnCell`; both now share one `sketchLink` helper.
- **No link to a missing record.**
  `RenderToDisk` passes `Render` a copy of the entries in which a discussed entry whose `decision-<slug>.md` is missing from the board directory is unflagged.
  Neither its README row nor its design doc then links the absent file.
  `Render` stays a pure function and still links every discussed entry it is given.

## Design decisions

- The existence check sits in `RenderToDisk`, the one render entry point that knows the board directory, rather than in a new `Render` parameter.
  The board write, the owner write and the Bolt replay all render through it,
  and each writes the record files before that render, so a freshly stored record is linked in the same write.
- A discussed entry with no body still gets no design doc; its README row carries the link.

## Tests

- `TestRenderDesignDocGoldens` gains a discussed entry with a dependency, pinning the record line's text and its place between the metadata and `Depends on:` lines.
- `TestRenderToDiskLinksOnlyAStoredDecisionRecord` renders a discussed entry with its record file and one without,
  and checks the README row and design doc of each: the first links the record, the second does not.

`internal/boardengine/doc.go`'s Decision record section states the design doc link and the missing-file rule.

---
round: 1
focus:
  - "internal/boardengine/render.go `withMissingRecordsUndiscussed`: does a Stat error other than not-exist (permission, I/O) wrongly drop a link to a record that exists, and is that the intended reading of 'missing'?"
  - "internal/boardengine/render.go `RenderToDisk` and `Render`: does any caller render to disk or to a string without going through `RenderToDisk`, so a flagged entry with a missing record still gets a link?"
  - "Goal bullet 2: does the Discussed flag survive a Sketch's promotion to a task and apply to every kind of discussed entry, so design page and README row both link the record for those too?"
  - "internal/boardengine/render.go `renderDesigns`: is the `Decision record:` line placed near the top, after the metadata line, and does an entry with a discussed flag but an empty body (no design doc) leave the README link as the only route?"
  - "internal/boardengine/render_test.go: do the new tests cover the design-doc link, the missing-file path and the promoted or non-Sketch case, without duplicating existing Sketch-link tests in the same file?"
  - "internal/boardengine/doc.go: does the changed prose match what the code does, including the claim that no board file links a missing record?"
---

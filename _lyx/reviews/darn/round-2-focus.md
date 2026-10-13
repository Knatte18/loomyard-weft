---
round: 2
focus:
  - "Webster fork prompt rendering in `internal/websterengine/render.go` and `beginbatch.go`: with a kick-start pack, does the prompt end with the one-turn read instruction (read all of this in one turn, in parallel, before any edit) followed by the pack inline as its last section, with no separate pack file written, no pack path passed into `RenderForkPrompt`, and no pointer line to another file? Check this against the task's board entry."
  - "Same rendering: does the batch-report section stay above that last section and still name the report as the last action, does a render without a pack end as before, does the recovery prompt take no pack, and does an empty pack render as the empty case? Check that docs (`doc.go`) and tests match the layout the code produces, and that no stencil, `.gitattributes` or test entry refers to a removed fragment."
---

The focus names where to look; severity and verdict stay the reviewer's call.

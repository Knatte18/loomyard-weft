# Card 6 — The Discussion-Write lead writes the relevant list

**Edit:**
- `internal/loomengine#DiscussionTable`
- `internal/loomengine#TestDiscussionTable`
- `contracts/stencils/loom/loom-template-discussion.md`

**Uses:**
- `internal/loomengine#DiscussionRelevant`

**Intent:**
Per D1, `relevant.md` becomes a required Discussion-Write output.

`DiscussionTable` appends `DiscussionRelevant(layout)` to the lead seat's `Outputs`, after the decision record and the support log, so the step's completion signal waits for it as it waits for the other two.
It adds a table value `relevant_path: DiscussionRelevant(layout)` beside `support_log_path`.
Its doc comment says the lead writes the decision record, the support log and the relevant list.
`resolveModelChoice` and the seat literal's shape are untouched, since the running sibling `engine-model-choice` rewrites the model resolution there.
The existing commit of the whole `_lyx/discussion/` directory carries the file, so no commit pathspec changes.

`loom-template-discussion.md`:
- Step 5 becomes "Write the three output files" and names `{{.relevant_path}}` beside the decision record and the support log, all three in the format `{{.specs_dir}}/loom/loom-discussion-spec.md` defines, which it does not restate (`PATTERN-producer-pointer-rule`).
- "What you may write" says the three files named in Step 5 and lists `{{.relevant_path}}`.
- The parent-review re-prompt bullet, and Step 6's gate re-prompt fix, each say a design change in that turn may update `{{.relevant_path}}` in the same turn.
The chair stencil includes this template whole, so it changes with it.

`TestDiscussionTable`'s chair outputs assertion wants `DiscussionDecisionRecord(layout)`, `DiscussionSupportLog(layout)` and `DiscussionRelevant(layout)`, in that order, over every advisor count and mode, and its value checks want `table.Values["relevant_path"]` to equal `DiscussionRelevant(layout)`.

**ImpactSummary:** one new lead output and table value in `DiscussionTable`, local to `internal/loomengine`; the advisors' seats and the darn stencils are untouched.

**Commit:** 6: Discussion-Write lead writes relevant.md

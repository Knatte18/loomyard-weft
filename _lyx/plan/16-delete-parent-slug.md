# Card 16 — delete-parent-slug

**Delete:**
- `internal/fabricengine#ParentSlug`
- `internal/fabricengine_test#TestParentSlug`

**Intent:**
`fabricengine.ParentSlug` derived a parent by matching the origin's `ParentBranch` against the pair worktrees present on this machine, which D4 forbids; card 15 moved every caller to `hubgeom.ParentOf`, whose own test covers the parent read.
Delete the function and its test, and drop its mention from the fabricengine package doc if one names it.

**ImpactSummary:** Removes one function and its test; card 15 removed its last caller.

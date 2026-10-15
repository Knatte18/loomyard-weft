# Card 2 — Declare the relevant list's path

**Create:**
- `internal/loomengine#DiscussionRelevant`

**Edit:**
- `internal/loomengine#DiscussionDecisionRecord`
- `internal/loomengine#TestDiscussionPaths`

**Intent:**
Add `DiscussionRelevant` to `internal/loomengine/config.go`, beside `DiscussionSupportLog`: it returns `relevant.md` joined onto `DiscussionDir(l)`, sharing its AnchorPath anchoring.
Its doc comment says it is the Discussion-Write lead's code-reading list, the plan writer's head start, and that per the Cwd Resolution Invariant no other package constructs this path (D1).

`DiscussionDecisionRecord`'s doc comment drops "the Plan producer's sole input from `_lyx/discussion/`" and calls the record the Plan producer's design input; its code is unchanged.

`TestDiscussionPaths` gains one assertion in its loop, over both the anchored and the unanchored row: `DiscussionRelevant(l)` equals `filepath.Join(wantDir, "relevant.md")`.
Its `//testtiming:keep` reason names the relevant list beside the decision record and support log paths.

**ImpactSummary:** one new exported accessor with no caller yet and a doc-comment change on `DiscussionDecisionRecord`; no behavior of an existing member changes.

**Commit:** 2: loomengine: declare the relevant list's path

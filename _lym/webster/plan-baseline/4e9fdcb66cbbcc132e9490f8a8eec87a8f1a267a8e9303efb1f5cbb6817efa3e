# Card 6 — discussion-approval-row

**Create:**
- `internal/loomrecipe#DiscussionApprovalRow`
- `internal/loomrecipe#PastDiscussionApproval`
- `internal/loomrecipe#discussionApprovalRow`
- `internal/loomrecipe#TestPastDiscussionApproval`

**Extends:**
- `internal/loomrecipe/doc.go`

**Uses:**
- `internal/loomrecipe#parseEmbeddedRecipe`
- `internal/loomrecipe#bouncerEngine`
- `internal/loomrecipe#Rows`
- `internal/loomrecipe#RetiredRows`

**Intent:**
Give D7's discussion approval row and D8's "past the approval" test one home, judged by a recipe's rows and never its name (`PATTERN-loom-contract`).

`discussionApprovalRow(parsed)` finds the row whose engine is `DiscussionWrite`.
It returns the row its `on_done` names when that row's engine is `bouncerEngine`, with true.
A recipe with no DiscussionWrite row, or whose DiscussionWrite row hands to a row that is not a Bouncer, answers false.
`DiscussionApprovalRow(recipe)` runs it over `parseEmbeddedRecipe(recipe)`.
An empty name reads as tapestry, and a name that selects no embedded recipe is refused, as the other accessors do.

`PastDiscussionApproval(recipe, currentRow)` answers hasRow false, and past false, for a recipe with no approval row.
Otherwise it is past when `currentRow`, first mapped through `RetiredRows(recipe)`, is a row of the recipe whose index in row order is greater than the index of every row whose segment is the approval row's segment.
Each of the following is not past:
- an empty `currentRow`, which is a run with no status file;
- a row before or inside that segment;
- a row the recipe does not hold.
It reads only the recipe's own rows, so a row inside another task's sub-run block is not reached, per the decision record's `Added after Discussion` entry.

`doc.go` gains one paragraph naming both accessors and the rule.

`TestPastDiscussionApproval` (tier 0) is a table.
- `DiscussionApprovalRow("tapestry")` is `Discussion-Bouncer`, true.
- `DiscussionApprovalRow("darn")` is false.
- On tapestry, `Preflight`, `Discussion-Write`, `Discussion-Bouncer`, `Discussion-Burler-Review`, `Discussion-Burler-Fix`, `""` and `No-Such-Row` are not past.
- On tapestry, `Drift-Baseline`, `Plan-Write` and `Code-Bouncer` are past, and the retired `Merge-In` maps to Parent-Align and is past.
- On darn and on pawl, any row answers hasRow false.
- An unknown recipe name returns an error.
- `discussionApprovalRow` over a `shedbuild.Parse`d fixture whose DiscussionWrite row's `on_done` names a non-Bouncer row answers false, and over a fixture with no DiscussionWrite row answers false.

**ImpactSummary:** Two new exported read-only accessors in loomrecipe, beside `EngineRow`. Nothing calls them yet: cards 7 to 9 add the callers.

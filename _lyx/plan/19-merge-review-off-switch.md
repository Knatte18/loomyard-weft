# Card 19 — merge-review-off-switch

**Create:**
- `plan:internal/loomrecipe#applyMergeReviewOff` -> `func applyMergeReviewOff(producers []shedengine.ProducerDef, off bool)`
- `plan:internal/loomrecipe#TestMergeReviewOff_RoutesMergeInToPublish` -> `func TestMergeReviewOff_RoutesMergeInToPublish(t *testing.T)`

**Edit:**
- `internal/loomrecipe#New`
- `internal/loomrecipe#Routing` -> `func Routing(reviewMaxBounces int, rowMaxBounces map[string]int, mergeReviewOff bool) (shedengine.Routing, error)`
- `internal/loomrecipe#newEscalationShed`
- `internal/loomrecipe#TestEscalation_PerchBudgetOneWithAutoAccept`
- `internal/loomrecipe#TestRouting_ProgressAt`
- `internal/loomrecipe#TestReviewBudget_AppliedByNewAndRouting`
- `internal/loomrecipe#TestReviewBudget_BelowOneRefused`
- `internal/loomcli#loomCLI.reviewBudget` -> `func (c *loomCLI) reviewBudget() (int, map[string]int, bool, error)`
- `internal/loomcli#loomCLI.recipeRouting`
- `internal/loomcli#loomCLI.wire`
- `internal/loomcli#TestWire_DefaultConfig`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:**
Add the Merge-Review off switch (D2).
`applyMergeReviewOff` rewrites `Merge-In`'s `OnDone` to `Publish` when `off` is true, as darn's 0 review budget rewrites past its perch.
`New` applies it with `env.MergeReviewOff` after the build, and `Routing` takes the same flag and applies it to its projection, so status progress and the bounce reporting see the graph the engine runs.
Bound: the switch skips only the review; Merge-In, the parent lock and the reroute still run, and its zero value keeps the review on.
`reviewBudget` also returns the loaded config's `SkipMergeReview`, the template's value when `loom.yaml` is unreadable.
`recipeRouting` and `wire`'s `SegmentBounces` projection pass the flag on.
Every test caller of `Routing` passes false.
`TestMergeReviewOff_RoutesMergeInToPublish` asserts that `New` and `Routing` with the flag route `Merge-In` straight to `Publish`, that the Merge-Review rows stay in the list unreached, and that without the flag `Merge-In` hands to `Merge-Bouncer`.

**ImpactSummary:** `Routing` and `loomCLI.reviewBudget` are re-signed with every caller on this card; nothing changes while `skip_merge_review` stays false.

**Commit:** 19: loom: skip_merge_review routes Merge-In straight to Publish

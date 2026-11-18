# Card 10 — recipes-pr-review-reroute

**Create:**
- `internal/loomrecipe#TestRecipe_PRReviewReroutesToCodeBouncer`

**Edit:**
- `//contracts/recipes/tapestry-recipe.yaml`
- `//contracts/recipes/darn-recipe.yaml`
- `//contracts/recipes/sampler-recipe.yaml`
- `//internal/loomrecipe/doc.go`
- `internal/loomrecipe#TestRouting_ProgressAt`

**Uses:**
- `internal/loomrecipe#TestRecipe_WebsterReroutesToPlanReview`

**Intent:** D4's edge: the PR-Review row of the tapestry, darn and sampler recipes declares `on_reroute: Code-Bouncer`, a row value (`PATTERN-shed-recipe-registry`).
The header comments that say PR-Review has no edge out name the reroute and say a send-back from `lym loom pr-findings` takes it.
These texts say PR-Review's Stuck always halts the run, and each now says its Stuck halts except a marked send-back's reroute to Code-Bouncer:
`internal/loomrecipe/doc.go`, the header comments of all three recipes, and the PR-Review row comment of all three.
Tapestry's header lists PR-Review among the rows whose Stuck halts the whole run, and it words the exception as it words Webster's.
`TestRouting_ProgressAt`'s tapestry-tail subtest wants the PR-Review row's reroute to be `loomshed.NameCodeBouncer`.
After the episode the run goes down its usual rows, Drift-Recheck, Describe, then PR-Review again.

Bound: the edge fires only on a marked send-back, which only `lym loom pr-findings` writes, and shed's reroute cap halts it past the cap for the parent.
Sampler's Code-Bouncer keeps its `sampler.code_review` entry gate, so with the review off the verb answers fix-here (card 8).

`TestRecipe_PRReviewReroutesToCodeBouncer`, tier0, beside `TestRecipe_WebsterReroutesToPlanReview` and built the same way, asserts for each of the three recipes that the built PR-Review row's `OnReroute` is Code-Bouncer.

**ImpactSummary:** Three recipe rows gain one routing edge; no other row or recipe changes.

**Commit:** `10: recipes-pr-review-reroute`

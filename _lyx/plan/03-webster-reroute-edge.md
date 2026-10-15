# Card 3 — webster-reroute-edge

**Create:**
- `plan:internal/loomrecipe#TestRecipe_WebsterReroutesToPlanReview` -> `func TestRecipe_WebsterReroutesToPlanReview(t *testing.T)`
- `plan:internal/loomcli#TestLoomRerouteCapWayForward_IsRowNeutral` -> `func TestLoomRerouteCapWayForward_IsRowNeutral(t *testing.T)`

**Edit:**
- `contracts/recipes/loom-recipe.yaml`
- `internal/loomcli#loomRerouteCapWayForward`
- `internal/shedadapters#TestBurlerReviewProducer_Call_RoundScan`
- `docs/overview.md`

**Uses:**
- `plan:internal/websterengine#ConflictRecordRel`

**Intent:**
Decision D3's recipe half and D5's recipe header: the loom graph gains the edge a Webster plan conflict travels, before any producer asks for it.

`contracts/recipes/loom-recipe.yaml`'s `Webster` row gains `on_reroute: Plan-Burler-Review`.
No row is renamed and no other edge changes.
The edge lets through only a Stuck verdict the Webster producer marks `Reroute`, which no producer does until card 4;
shed's reroute cap (three per row since its latest Done, and Webster is Done only when its whole run finishes) bounds it, a reroute spends no bounce budget, and `internal/shedengine`'s validator already requires the target to exist.
The header comment's escalate-to-a-human paragraph notes that Webster's Stuck reroutes to Plan-Burler-Review when begin-batch left a new conflict record, and halts otherwise.
`Plan-Burler-Review`'s fasit `instructions` gain: when `.lyx/webster/plan-conflict.md` exists, Webster's begin-batch refused the approved plan against the current tree, each blocking finding it lists is a finding of this round, fixed by editing only cards whose batch has not begun.
The rerouted review continues the `plan` run directory as the next round of the approved episode: `BurlerReviewProducer.Call` spawns a reviewer for the successor of a round carrying a Bouncer verdict, so no producer change is needed.

`loomRerouteCapWayForward` in `internal/loomcli/wiring.go` becomes row-neutral: `run "lyx loom resume" in the task worktree once the cause the halt's reason quotes is cleared: for a parent-branch reroute, once the parent branch has settled; for a Webster plan conflict, once the named pending cards are edited by hand to match the tree`.
Its comment says it serves every row's cap halt.
`docs/overview.md`'s execution-stack paragraph, which names `Publish`, `PR-Gate` and `Finalize` as the rows that reroute to `Merge-In`, gains that `Webster` reroutes to `Plan-Burler-Review` when begin-batch left a new plan conflict.

Tests:
the new `TestRecipe_WebsterReroutesToPlanReview` in `internal/loomrecipe/recipe_test.go` (package budget has room for one) builds the recipe through `New` and asserts the `Webster` row's `OnReroute` is `loomshed.NamePlanBurlerReview`, then parses `recipes.LoomRecipe` with `shedbuild.Parse` and asserts the `Plan-Burler-Review` row's `config.profile.fasit.instructions` string contains `websterengine.ConflictRecordRel()`, pinning the recipe's path text to the `websterengine` declaration;
`TestRecipe_StructuralCheckHasNoFindings` keeps proving the recipe validates with the new edge.
`TestBurlerReviewProducer_Call_RoundScan` gains the row "the successor of a converged round, as a Webster reroute after an approved episode, is reviewed": round 2's pair plus a `CONVERGED` verdict and ledger, wanting round 3 reviewed.
The new `TestLoomRerouteCapWayForward_IsRowNeutral` in `internal/loomcli/wiring_test.go` asserts the clause names `lyx loom resume`, the parent-branch cause and the plan-conflict hand edit of the named pending cards, so neither cause reads as the only one.

**ImpactSummary:** One recipe edge and one fasit paragraph, inert until card 4's producer asks for the reroute; the cap clause text changes for every loom row's reroute-cap halt, which today only Publish, PR-Gate and Finalize reach, and no test asserts its old wording.

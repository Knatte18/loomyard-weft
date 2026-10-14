# Card 2 — zero-round perch: the Bouncer settles at once

**Edit:**
- `internal/loomrecipe#New`
- `internal/loomrecipe#Routing` -> `func Routing(reviewMaxBounces int, rowMaxBounces map[string]int) (shedengine.Routing, error)`
- `internal/loomrecipe#applyReviewBudget`
- `internal/loomrecipe#checkReviewBudget`
- `internal/loomrecipe#assertReviewBudget`
- `internal/loomrecipe#TestReviewBudget_AppliedByNewAndRouting`
- `internal/loomrecipe#newEscalationShed`
- `internal/loomrecipe#TestEscalation_PerchBudgetOneWithAutoAccept`
- `internal/loomrecipe#TestRouting_ProgressAt`
- `internal/loomrecipe/doc.go`
- `internal/shedrecipe#Env`
- `internal/shedrecipe#bouncerEntry`
- `internal/shedrecipe#TestBouncerEntry_SkipSeam`
- `internal/loomcli#loomCLI.wire`
- `internal/loomcli#loomCLI.recipeRouting`
- `internal/loomcli#loomCLI.reviewBudget` -> `func (c *loomCLI) reviewBudget() (int, map[string]int, error)`
- `internal/loomcli#perchMaxBounces`
- `internal/loomcli#TestWire_DefaultConfig`
- `internal/loomcli#TestWire_ReviewKeysReachTheEnv`
- `internal/loomcli#TestWire_MergeReviewSeams`
- `contracts/recipes/loom-recipe.yaml`

**Delete:**
- `internal/loomrecipe#applyMergeReviewOff`

**Rename:**
- `internal/loomrecipe#TestMergeReviewOff_RoutesMergeInToPublish` -> `plan:internal/loomrecipe#TestZeroBudget_BouncerSettlesAtOnce`
- `internal/loomrecipe#TestReviewBudget_BelowOneRefused` -> `plan:internal/loomrecipe#TestReviewBudget_NegativeRefused`

**Create:**
- `plan:internal/loomrecipe#stampZeroRounds` -> `func stampZeroRounds(recipe *shedbuild.Recipe, segments map[string]string, fallback int, rowMaxBounces map[string]int)`

**Uses:**
- `internal/loomrecipe#reviewSegments`
- `internal/loomrecipe#testEnv`
- `internal/loomrecipe#NewDarn`

**Intent:**
The routing half of the record's "Zero-round perch: the Bouncer settles at once" entry, ahead of the config change in card 4: a review perch whose resolved budget is 0 keeps its writer → Bouncer route, and its Bouncer settles approved at once with no review round, running its approve and commit seams.
A budget of 0 now means zero rounds, and an absent per-perch entry, not a 0 one, takes the run-wide budget.

`internal/loomrecipe`:

- `checkReviewBudget` refuses a budget below 0, not below 1: `loomrecipe: <name> = <v>; want 0 or more (set review_max_bounces in loom.yaml)`.
  `New` and `Routing` still call it first on the run-wide value, so a run-wide 0 passes and gives every perch without its own entry zero rounds, per the settled design's point (2).
- `applyReviewBudget` gives a segment's rows the Bouncer's `rowMaxBounces` entry whenever the map holds one, 0 included, and `fallback` only when it holds none; a negative entry is still refused naming the row.
  `NewDarn` and `DarnRouting` call it with a nil map and a positive budget, so darn is unchanged.
- `stampZeroRounds` resolves each review segment's Bouncer budget the same way and, for each one at 0, sets `zero_rounds: true` in that Bouncer row's parsed `Config` map, as `applyVerifyAttempts` stamps the verify gate's attempts.
  `New` computes the segments before the build and calls it on the parsed recipe before `shedbuild.NewShedFrom`; `Routing` builds no engine and does not call it: a zero-round segment's rows carry `MaxBounces` 0, which the progress projection reads as `shedengine`'s inherited budget, so status progress shows the shed's or engine's default bounce budget for a perch at 0, a display gap only, since a zero-round Bouncer never bounces.
- `applyMergeReviewOff` is deleted with its two calls, and `Routing` loses its `mergeReviewOff` argument: a Merge-Review at 0 runs Merge-In → Merge-Bouncer → Publish with the Bouncer settling at once, which replaces routing Merge-In's Done to Publish.
- `doc.go` and the `New` and `Routing` doc comments say: an entry present in the per-perch map wins, 0 included, and a 0 budget stamps `zero_rounds` on the perch's Bouncer row; the paragraph on the run-wide value refused below 1 says below 0.
  The `Routing` doc's claim that the projection never reports the engine default for a review row gains its exception: a zero-round segment's rows carry `MaxBounces` 0, which the progress projection reads as `shedengine`'s inherited budget.

`internal/shedrecipe`:

- `Env.MergeReviewOff` is removed; the `RowMaxBounces` doc comment says a missing entry takes `ReviewMaxBounces` and an entry of 0 gives the perch zero rounds.
- `bouncerEntry` reads an optional boolean `zero_rounds` key, default false, through `configBool` as it reads `blocking_escalates`.
  True sets the Bouncer's `Skip` seam to a closure answering true, over any `skip_seam` the row names, so `Call` settles through `approveWithoutReview`: Approve, then Commit, then Done, with no seed or judge spawn and no carry-over.
  The doc comment lists the key, and says the recipe declares it on no row: loomrecipe stamps it.
- `TestBouncerEntry_SkipSeam` gains two subtests: `ZeroRoundsApprovesAtOnce`, a row over `minimalBouncerConfig` with `zero_rounds: true`, `approve_seam: plan`, `commit_seam: plan` and `skip_seam: rework-exempt`, and `env.SkipPlanReview` answering false, whose `Call` returns Done with `env.ApprovePlan` and `env.CommitPlan` each called once and no shuttle start; and `NonBooleanZeroRoundsFails`, whose construction error names `zero_rounds`.

`internal/loomcli`, which keeps loom.yaml's present meaning until card 4 changes it:

- `perchMaxBounces` puts a perch's entry in the map only when its loom.yaml key is above 0, since 0 still means inherit there, and puts `Merge-Bouncer: 0` when `cfg.SkipMergeReview` is true.
- `loomCLI.wire` drops the `MergeReviewOff` field and the third `loomrecipe.Routing` argument in its `SegmentBounces` closure; `loomCLI.reviewBudget` drops its skip result, and `loomCLI.recipeRouting` calls `loomrecipe.Routing(budget, rowBudgets)`.
- `TestWire_DefaultConfig`'s "merge review stays on by default" subtest asserts `c.env.RowMaxBounces` has no `Merge-Bouncer` entry of 0 under the template, and its `loomrecipe.Routing` call drops the `false`.
- `TestWire_ReviewKeysReachTheEnv` expects `RowMaxBounces` without the Discussion-Bouncer entry, since its key is 0, and with `Merge-Bouncer: 0` from the seeded `skip_merge_review: "true"`, replacing its `MergeReviewOff` assertion.
- `TestWire_MergeReviewSeams` asserts `c.env.RowMaxBounces[Merge-Bouncer]` is 0 from `skip_merge_review`, replacing its `MergeReviewOff` assertion.

`contracts/recipes/loom-recipe.yaml`'s comment at the per-perch budget (falling back to `review_max_bounces` where that is 0) and at Merge-In (`skip_merge_review` routes Merge-In's Done past the review) say instead: an absent per-perch budget takes `review_max_bounces`, and a perch at 0 keeps its route and its Bouncer settles at once.

Tests in `internal/loomrecipe/reviewbudget_test.go`:

- `assertReviewBudget` expects a row's `MaxBounces` to be its Bouncer's entry whenever `rowBudgets` holds one, 0 included.
- `TestReviewBudget_AppliedByNewAndRouting` replaces its "zero entries take the run-wide budget" row with "an absent entry takes the run-wide budget", keeps the others, and adds "a zero entry is zero rounds" with `Plan-Bouncer: 0`.
- `TestZeroBudget_BouncerSettlesAtOnce`, renamed from `TestMergeReviewOff_RoutesMergeInToPublish`, is a table over `New`: one row per Bouncer at 0 through `RowMaxBounces`, and a row with `ReviewMaxBounces` 0 and no entries.
  Each row finds the built Bouncer producers and `Call`s each one at 0, asserting Done and no shuttle start, which proves the stamp reached the row; the seam order of the settle is `TestBouncerEntry_SkipSeam`'s to pin.
  It also asserts, for `New` and for `Routing` with the same budgets, that Discussion-Write, Plan-Write, Webster and Merge-In still name their Bouncer as `OnDone` and every Merge-Review row stays in the list.
- `newEscalationShed`, `TestEscalation_PerchBudgetOneWithAutoAccept` and `TestRouting_ProgressAt` drop the third `Routing` argument and change nothing else.
- `TestReviewBudget_NegativeRefused`, renamed from `TestReviewBudget_BelowOneRefused`, asserts `Routing(-1, nil)` and `New` with `ReviewMaxBounces` -1 refuse naming `ReviewMaxBounces`, `Routing(0, nil)` passes, and a negative per-perch entry is refused naming its row.

**ImpactSummary:** the loom recipe builder, its routing projection and the Bouncer registry entry gain zero-round perches and lose the merge-review route switch; loomcli's wiring adapts so loom.yaml keeps its present meaning until card 4.

**Commit:** 2: loomrecipe: a perch at budget 0 settles in its Bouncer at once

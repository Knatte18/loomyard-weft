# Card 7 — budgets-by-segment

**Create:**
- `plan:internal/loomrecipe#segmentBudgets` -> `func segmentBudgets(segments map[string]string, env shedrecipe.Env, wayForward string) (map[string]int, error)`
- `plan:internal/loomrecipe#checkSegmentBudgets` -> `func checkSegmentBudgets(segments map[string]string, budgets map[string]int, wayForward string) error`
- `plan:internal/loomcli#loomSegmentBudgets` -> `func loomSegmentBudgets(cfg loomengine.Config) map[string]int`

**Edit:**
- `internal/loomrecipe#New`
- `internal/loomrecipe#Routing` -> `func Routing(budgets map[string]int) (shedengine.Routing, error)`
- `internal/loomrecipe#NewDarn`
- `internal/loomrecipe#DarnRouting` -> `func DarnRouting(budgets map[string]int) (shedengine.Routing, error)`
- `internal/loomrecipe#applyReviewBudget` -> `func applyReviewBudget(producers []shedengine.ProducerDef, segments map[string]string, budgets map[string]int)`
- `internal/loomrecipe#stampZeroRounds` -> `func stampZeroRounds(recipe *shedbuild.Recipe, segments map[string]string, budgets map[string]int)`
- `internal/loomrecipe#applyDarnReviewBudget` -> `func applyDarnReviewBudget(producers []shedengine.ProducerDef, budgets map[string]int) error`
- `internal/loomcli#loomCLI.recipeRouting`
- `internal/loomcli#loomCLI.reviewBudget` -> `func (c *loomCLI) reviewBudget() (map[string]int, error)`
- `internal/loomcli#loomCLI.darnReviewBudget` -> `func (c *loomCLI) darnReviewBudget() (map[string]int, error)`
- `internal/loomcli#loomCLI.wire`
- `internal/loomcli#loomCLI.wireDarn`
- `internal/loomcli#TestWire_DefaultConfig`
- `internal/loomcli#TestSpecFor_FillsRunIdentityForSelf`
- `internal/loomrecipe#newEscalationShed`
- `internal/loomrecipe#TestEscalation_DarnPerchRuleContinuesThenHardCapAcceptReachesDescribe`
- `internal/loomrecipe#TestEscalation_PerchBudgetOneWithAutoAccept`
- `internal/loomrecipe#TestRouting_ProgressAt`
- `internal/loomrecipe#TestReviewBudget_AppliedByNewAndRouting`
- `internal/loomrecipe#TestZeroBudget_BouncerSettlesAtOnce`
- `internal/loomrecipe#TestReviewBudget_NegativeRefused`
- `internal/loomrecipe#TestBounceRouting_BudgetExhaustionBlocks`
- `internal/loomrecipe#TestNewDarn_Routing`
- `internal/loomrecipe#TestDarnVerifyBudget`
- `internal/loomrecipe#TestDarnRouting`
- `internal/loomrecipe/doc.go`

**Delete:**
- `internal/loomrecipe#resolvedBudget`
- `internal/loomrecipe#checkReviewBudget`
- `internal/loomrecipe#checkDarnReviewBudget`

**Uses:**
- `internal/loomcli#loomSegmentPerches`
- `internal/loomcli#darnReviewSegment`
- `internal/loomrecipe#reviewSegments`

**Intent:** D8: `internal/loomrecipe`'s budget stamping and routing are keyed by segment, `New` and `NewDarn` read each review segment's budget from `env.Segments`, and `Routing` and `DarnRouting` take per-segment budgets alone.

`internal/loomrecipe`:
- `segmentBudgets` maps each review segment of `segments` (segment → Bouncer row, from `reviewSegments`) to `env.Segments[segment].MaxBounces`.
  A review segment with no `env.Segments` entry is refused naming the segment and `wayForward`, e.g. `loomrecipe: review segment "S" has no Env.Segments entry; way forward: add the segment's row to the loom wiring's segment-to-perch table`: `New` passes that segment-to-perch-table text and `NewDarn` the text naming the darn wiring's `Darn-Review` entry.
  A negative budget is refused naming the segment, its `MaxBouncesKey` and `ConfigFile`, e.g. `loomrecipe: bounce budget of review segment "Plan-Review" = -1 from plan_max_bounces in loom.yaml; want 0 or more; way forward: set plan_max_bounces in loom.yaml`; for `Darn-Review` the way forward keeps today's text from `checkDarnReviewBudget`, `run "lyx config darn --set review_max_bounces=<n>" from the prime, then "lyx loom resume"`, after naming the segment, `review_max_bounces` and `darn.yaml`.
- `checkSegmentBudgets` is the routing side: a review segment `budgets` lacks is refused naming the segment and `wayForward`, and a negative entry is refused naming the segment with generic text, as `applyReviewBudget`'s refusal words it today, since `loomrecipe` holds no segment-to-perch table: for a loom segment the way forward `set the perch's <perch>_max_bounces, or review_max_bounces, in loom.yaml`, and for `Darn-Review` the same runnable `lyx config darn --set review_max_bounces=<n>` then `lyx loom resume` way forward as `segmentBudgets`, after naming `review_max_bounces` and `darn.yaml`.
- `New` computes `segmentBudgets` before the build, `stampZeroRounds` stamps `zero_rounds` on the Bouncer of every segment whose budget is 0, and `applyReviewBudget` sets each review segment row's `MaxBounces` from `budgets[segment]` after the build; `applyReviewBudget` and `stampZeroRounds` read the map alone, since every key is checked first, and `resolvedBudget` and the run-wide fallback go.
- `Routing(budgets)` checks with `checkSegmentBudgets` (way forward: add the segment's row to the loom wiring's segment-to-perch table) and applies the same stamping as `New`, so the projection equals the build for one budget map.
- `NewDarn` takes `Darn-Review`'s budget from `env.Segments` through `segmentBudgets` and `DarnRouting(budgets)` checks with the way forward to add the `Darn-Review` budget entry the darn wiring builds from `darn.yaml`'s `review_max_bounces`; `applyDarnReviewBudget` reads `budgets["Darn-Review"]`, keeps rewriting `Darn-Write`'s Done to `Merge-In` at 0, and otherwise applies `applyReviewBudget`.
- `checkReviewBudget` and `checkDarnReviewBudget` go with their callers on this card.
- `doc.go`'s budget paragraphs are rewritten: budgets come from `Env.Segments` keyed by segment for `New` and `NewDarn` and from the per-segment map for `Routing` and `DarnRouting`, with the missing-segment and negative refusals.

`internal/loomcli`:
- `loomSegmentBudgets` maps each `loomSegmentPerches` row to `cfg.Caps(perch).Soft`, the value `loomSegmentSettings` puts in `MaxBounces`, so the status path and the build share one table and one resolution.
- `reviewBudget` returns `loomSegmentBudgets` of `c.cfg` when loaded, else of a fresh `loomengine.LoadConfig`, else of the template with today's warning, so the status and pause path keeps its degrade-to-template read and gains no model resolution.
- `darnReviewBudget` returns `{darnReviewSegment: review_max_bounces}` from the same three sources.
- `recipeRouting` passes those maps to `DarnRouting` and `Routing`; `wire`'s `SegmentBounces` closure passes `loomSegmentBudgets(loomCfg)` and `wireDarn`'s passes `{darnReviewSegment: cfg.ReviewMaxBounces}`.
`wire` still fills `ReviewMaxBounces` and `RowMaxBounces`, now unread, until card 8.

Tests:
- `TestWire_DefaultConfig`'s segment-bounces subtest builds its routing with `loomrecipe.Routing(loomSegmentBudgets(c.cfg))` and asserts every review row of the built shed carries the `MaxBounces` the status path's `c.recipeRouting()` projects for one `loom.yaml`, extending today's single-row check to every row whose segment holds a Bouncer.
- `TestSpecFor_FillsRunIdentityForSelf` runs on a receiver with no `Segments` and no config file; it gains the assertion that each review row's `MaxBounces` in `spec.Routing` equals its perch's `Caps(perch).Soft` over the loom template (the template's `review_max_bounces` for Discussion, Plan and Webster, and its own `merge_max_bounces` for Merge), so the status path still degrades to the template.
- `TestReviewBudget_AppliedByNewAndRouting` and `TestZeroBudget_BouncerSettlesAtOnce` set budgets on `env.Segments` for `New` and pass the same map to `Routing`; a zero entry still gives that perch zero rounds.
- `TestReviewBudget_NegativeRefused`'s existing assertions change: `Routing(-1, nil)` and the `ReviewMaxBounces` substring go with the run-wide fallback, and its negative-row refusal now names the segment, not `Webster-Bouncer`; it gains rows: `New` with a negative `Plan-Review` budget names `Plan-Review`, `plan_max_bounces` and `loom.yaml`; `New` with `Webster-Review` deleted from `env.Segments` names the segment and the segment-to-perch table; `Routing` with a map lacking `Merge-Review` names the segment and the table; `Routing` with a negative entry names the segment.
- `TestDarnRouting` passes `{"Darn-Review": n}`, keeps the 0 edge-rewrite row, turns its `DarnRouting(-1)` row into `{"Darn-Review": -1}`, refused naming the segment and `review_max_bounces` in `darn.yaml` with the `lyx config darn --set review_max_bounces=<n>` then `lyx loom resume` way forward, and gains a row where the map lacks `Darn-Review`, refused naming `Darn-Review` and the darn wiring's budget entry.
- `TestNewDarn_Routing` and `TestDarnVerifyBudget` set `env.Segments["Darn-Review"].MaxBounces` (the negative row with `MaxBouncesKey` `review_max_bounces`, `ConfigFile` `darn.yaml`, refused naming all three and asserting the `lyx config darn --set review_max_bounces=<n>` then `lyx loom resume` way forward); `TestDarnVerifyBudget` gains a row with `Darn-Review` deleted from `env.Segments`, refused by `NewDarn` before any row builds, naming `Darn-Review` and the darn wiring's `Darn-Review` entry.
- `newEscalationShed`, the two escalation tests, `TestRouting_ProgressAt` and `TestBounceRouting_BudgetExhaustionBlocks` set or read budgets through `env.Segments` and a segment-keyed map; `TestBounceRouting_BudgetExhaustionBlocks`'s doc comment, which says Discussion-Bouncer's budget is the fixture's `Env.ReviewMaxBounces`, names `Env.Segments["Discussion-Review"].MaxBounces` instead.

**ImpactSummary:** loomrecipe's two routing projections re-signed with their callers in loomcli's status and wire paths and loomrecipe's tests; New and NewDarn read budgets from Env.Segments with two new refusals; the status path keeps its degrading reads.

**Commit:** 7: loomrecipe: key review budgets by segment

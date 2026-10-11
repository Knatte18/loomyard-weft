# Card 9 — perch-budget-wiring

**Create:**
- `plan:internal/loomcli#perchMaxBounces` -> `func perchMaxBounces(cfg loomengine.Config) map[string]int`
- `plan:internal/loomcli#perchAutoAccept` -> `func perchAutoAccept(cfg loomengine.Config) map[string]bool`
- `plan:internal/loomrecipe#TestEscalation_PerchBudgetOneWithAutoAccept` -> `func TestEscalation_PerchBudgetOneWithAutoAccept(t *testing.T)`

**Edit:**
- `internal/shedrecipe#Env`
- `internal/shedrecipe#bouncerEntry`
- `internal/testkit/envkit#nilLegal`
- `internal/loomrecipe#New`
- `internal/loomrecipe#Routing` -> `func Routing(reviewMaxBounces int, rowMaxBounces map[string]int) (shedengine.Routing, error)`
- `internal/loomrecipe#reviewSegments` -> `func reviewSegments(source []byte) (map[string]string, error)`
- `internal/loomrecipe#applyReviewBudget` -> `func applyReviewBudget(producers []shedengine.ProducerDef, segments map[string]string, fallback int, rowMaxBounces map[string]int) error`
- `internal/loomrecipe#applyDarnReviewBudget`
- `internal/loomrecipe#TestReviewBudget_AppliedByNewAndRouting`
- `internal/loomrecipe#TestReviewBudget_BelowOneRefused`
- `internal/loomrecipe#TestRouting_ProgressAt`
- `internal/loomrecipe#newEscalationShed`
- `internal/loomrecipe/doc.go`
- `internal/loomcli#loomCLI.wire`
- `internal/loomcli#loomCLI.recipeRouting`
- `internal/loomcli#loomCLI.reviewBudget` -> `func (c *loomCLI) reviewBudget() (int, map[string]int, error)`
- `internal/loomcli#TestWire_DefaultConfig`
- `internal/loomcli#TestWire_ReviewKeysReachTheEnv`
- `contracts/recipes/loom-recipe.yaml`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:**
D10's wiring half and D11's flag: each loom perch's rows carry the perch's own resolved budget, and its Bouncer is told its auto-accept flag.

`shedrecipe.Env` gains `RowMaxBounces map[string]int` and `RowAutoAccept map[string]bool`, both keyed by a review segment's Bouncer row name, as `RowBouncerModels` is; a nil map and a missing entry mean 0 (take `ReviewMaxBounces`) and false.
`envkit`'s `nilLegal` admits both as nil.
`bouncerEntry` sets `BouncerConfig.AutoAccept` from `Env.RowAutoAccept[name]`.

`loomrecipe`: `reviewSegments` maps each review segment to its Bouncer row, and `applyReviewBudget` sets every row of a segment (the Bouncer, the review row and the fix row) to that Bouncer's `rowMaxBounces` entry when it is above 0 and to `fallback` otherwise.
A negative entry, which `LoadConfig` already refuses upstream, returns a fail-loud wiring-guard error naming the Bouncer row, in `refusal-spec.md`'s grouped wiring-guard sense, with no row of its own.
`New` and `Routing` apply `Env.ReviewMaxBounces` (still refused below 1 as today, naming it) and the row map, so the engine and the routing projection hold the same per-perch value; the darn recipe passes no row map, so the Darn-Review perch keeps `darn.yaml`'s budget.
A budget of N gives N judged Burler rounds, and under a budget of 1 a fix gate exhausted in round 1 leaves no judged round, so the Bouncer's re-bounce Stuck meets Shed's generic budget block and the run halts for a human, whatever the flag.

`loomcli`: `perchMaxBounces` and `perchAutoAccept` map `loom.yaml`'s per-perch keys onto the Discussion, Plan and Webster Bouncer rows' names; `loomCLI.wire` fills `Env.RowMaxBounces` and `Env.RowAutoAccept` from them and builds the `SegmentBounces` seam's routing from the same map, so the Bouncer's bounce seam and the routing read one per-perch value; `reviewBudget` returns the run-wide budget and the row map, from the loaded config or, for the lightweight verbs, from `loom.yaml` read again or the template, and `recipeRouting` passes both to `Routing`.
`loom-recipe.yaml`'s header and `internal/loomrecipe/doc.go` say the review segments' budget is the perch's `<perch>_max_bounces`, falling back to `review_max_bounces`.

Tests:
`TestReviewBudget_AppliedByNewAndRouting` gains rows where one perch's budget is set and only that segment's three rows carry it, while the others carry the run-wide budget; `TestReviewBudget_BelowOneRefused` passes the new `Routing` argument.
`TestRouting_ProgressAt` and `newEscalationShed` pass no row map; with every new key at its default the existing escalation and routing tests pass unchanged, which pins that defaults keep today's routing, budgets and escalations.
`TestEscalation_PerchBudgetOneWithAutoAccept` drives the sequence fixture with Webster-Bouncer's budget at 1 and its flag true and a judge ruling CONTINUE: in one row the Webster-Review segment runs exactly one review and one fix call and Webster-Bouncer settles Done with an automatic accept on disk, and the run reaches Describe; in the other the fake fix half's gate fails in round 1 and the run halts blocked with no circling decision and no Webster-Bouncer Done.
`TestWire_ReviewKeysReachTheEnv` asserts set per-perch keys reach both maps under the Bouncer rows' names, and `TestWire_DefaultConfig` calls `Routing` with the row map.
The `internal/loomrecipe` row of `test-budget.yaml` rises by the new test.

**ImpactSummary:** `Routing` and the recipe budget helpers are re-signed with their loomcli and test callers; with every new key at its default the built graph and the status projection are unchanged.

**Commit:** 9: loom: each perch takes its own budget and auto-accept flag

# Card 4 — Review segments take their bounce budget from `loom.yaml`

**Edit:**
- `contracts/recipes/loom-recipe.yaml`
- `internal/shedrecipe/recipe.go`
- `internal/loomrecipe/loomrecipe.go`
- `internal/loomrecipe/doc.go`
- `internal/loomcli/wiring.go`
- `internal/loomcli/arm.go`
- `internal/loomcli/reject.go`
- `internal/loomrecipe/fixture_test.go`
- `internal/loomrecipe/shape_test.go`
- `internal/loomrecipe/recipe_test.go`
- `internal/loomrecipe/resume_test.go`
- `internal/loomcli/wiring_test.go`
- `internal/testkit/envkit/envkit.go`

**Create:**
- `internal/loomrecipe/reviewbudget_test.go`

**Uses:**
- `internal/loomengine/config.go`
- `internal/loomrecipe/bouncerrows.go`

**Intent:**
D3 and D4: the bounce budget of every Bouncer review segment's rows is `review_max_bounces`, applied by the loom recipe assembly, and the `Bounces` seam reads the same budget Shed's engine blocks on.

- `loom-recipe.yaml`: drop `max_bounces` from the six rows of the Discussion-Review, Plan-Review and Webster-Review segments, and add one comment saying their budget comes from `loom.yaml`'s `review_max_bounces`.
  `PR-Gate` keeps its literal `max_bounces`.
- `shedrecipe.Env` gains `ReviewMaxBounces int`, a run-wide value read by `loomrecipe`, documented beside the `Review*` fields.
- `loomrecipe`: a review segment is a segment holding a row whose engine is `Bouncer` (reuse `bouncerEngine`).
  An unexported helper sets `MaxBounces` to the budget on every row of a review segment.
  `New` applies `env.ReviewMaxBounces` to the built producer list before returning, and refuses a value below 1 as a wiring error naming `ReviewMaxBounces`.
  `Routing` becomes `Routing(reviewMaxBounces int)` and applies the same helper, refusing a value below 1 the same way, so a routing projection never reports the engine default for a review row.
  Update both doc comments and the package doc where they describe budgets.
- `loomcli`: `wire` fills `env.ReviewMaxBounces` from `loomCfg.ReviewMaxBounces`;
  `segmentBounces` takes the budget and calls `loomrecipe.Routing(budget)`;
  `loadRouting` in `arm.go` and the reject verb in `reject.go` pass `c.cfg.ReviewMaxBounces`.
- The shared `envkit.FullEnv` sets `ReviewMaxBounces` once (`PATTERN-testkit`), so every `Env` built from it passes the new positivity check;
  per-package fixtures set it only where a test needs a different value, and any other fixture that builds an `Env` outside the kit sets it too;
  `resume_test.go`'s budget-exhaustion test takes its budget from the fixture's `ReviewMaxBounces` rather than a recipe literal, and its comments stop claiming the row declares `max_bounces`;
  `recipe_test.go`'s perch check keeps asserting equal `Segment` and `MaxBounces` over the built rows.

Tests:
- `reviewbudget_test.go`: with a budget of 7, every Discussion-Review, Plan-Review and Webster-Review row built by `New` and projected by `Routing` carries `MaxBounces` 7, `PR-Gate` keeps the recipe's value, and `New` and `Routing` refuse 0.
- `wiring_test.go`: `TestWire_BouncerSlugAndSegmentBounces` expects the configured budget, and a new assertion pins that, for a review Bouncer row, the budget `env.SegmentBounces` reports equals the `MaxBounces` of that row in the `Shed` that `loomrecipe.New` builds from the same wiring.

**ImpactSummary:** Every caller of `loomrecipe.Routing` and every `Env` built for `loomrecipe.New` changes; the review rows' effective budget stays 5 under the template default.

**Commit:** 4: Review segments take their bounce budget from loom.yaml

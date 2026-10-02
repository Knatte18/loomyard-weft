# Card 10 — rework-routes-through-plan-review

**Edit:**
- `internal/shedrecipe/entries_bouncer.go`
- `internal/shedrecipe/entries_bouncer_test.go`
- `internal/shedrecipe/recipe.go`
- `internal/loomcli/wiring.go`
- `internal/loomcli/wiring_test.go`
- `internal/loomrecipe/shape_test.go`
- `internal/loomrecipe/recipe_test.go`
- `internal/loomrecipe/approveseam_test.go`
- `internal/loomrecipe/fixture_test.go`
- `internal/loomcli/wiring_rework_integration_test.go`

**Prosa:**
- `contracts/recipes/loom-recipe.yaml`
- `contracts/specs/shed-recipe-spec.md`
- `docs/overview.md`

**Uses:**
- `plan:internal/loomshed#PlanReviewSkippable`
- `internal/shedadapters/bouncer.go`

**Intent:**
Route a rework generation through Plan-Review's own segment (D7): `PR-Rework.on_done` becomes `Plan-Bouncer`, so the generation walks Plan-Review → Batchifier → Webster → Webster-Review → Describe → Publish → PR-Gate, and Plan-Bouncer skips its judge for an exempt generation.

- `bouncerEntry` accepts an optional `skip_seam` config key, parsed beside `commit_seam` and `approve_seam` (Shed Recipe Registry and Recipe-Format Sole-Parser invariants).
  Its one accepted value, `rework-exempt`, resolves to a new `Env.SkipPlanReview func() (bool, error)`, guarded by `requireSeam`; absent leaves `BouncerConfig.Skip` nil; any other value is a construction error naming the accepted value.
- `wire()` fills `Env.SkipPlanReview` with `loomshed.PlanReviewSkippable` over the same rework deps PR-Rework gets (`committedAnchoredReader` included); building it opens nothing.
- `loom-recipe.yaml`: `PR-Rework.on_done: Plan-Bouncer`; Plan-Bouncer gains `skip_seam: rework-exempt` with a comment stating the bound (the seam's inputs are agent-written, the overview's `first_card` and a swept `record.json`, so the live committed cards are re-classified and a stale or forged record can at most skip a generation whose live cards are themselves exempt; generation 0 never skips; a seam error falls back to a real review); the header comment and the PR-Rework row comment describe generation routing in place of "Rework cards skip Plan-Review because the findings are operator-authored".
  No row is renamed or removed.
- `shed-recipe-spec.md` documents `skip_seam` beside `commit_seam`.
- `docs/overview.md`'s rework paragraph names the route through Plan-Review and the exempt-generation skip.

The Plan-Review bounce budget resets on each entry: a Done by any row of a segment ends its rows' bounce episode (`internal/shedengine`), and the skip's Done is such a Done.

Tests: the shape and recipe tests pin `PR-Rework.on_done` as `Plan-Bouncer`; a recipe test asserts `skip_seam` sits on Plan-Bouncer only (beside the approve-seam test); `entries_bouncer_test.go` covers the absent key, `rework-exempt` with a nil `Env.SkipPlanReview` failing construction, and an unknown value failing construction; `wiring_test.go` asserts `wire()` fills `SkipPlanReview`; the loomrecipe fixture fills the new Env field.
The rework integration test carries a round on through the new route's first decision: after a round whose new generation is all-`Prosa` on `.md` files, `Env.SkipPlanReview` over the real fabric pair answers true, and after a round whose generation carries an `Edit` card it answers false.

**ImpactSummary:** one new Bouncer config key and Env field, one recipe edge and one row key; every other row and the recipe's other edges are unchanged.

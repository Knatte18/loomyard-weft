# Card 10 — drift-verify-before-review

**Edit:**
- `contracts/recipes/tapestry-recipe.yaml`
- `internal/loomrecipe/doc.go`
- `internal/loomrecipe#wantProducerTable`
- `internal/loomrecipe#TestRowNames_FollowEachRecipesRowOrder`
- `internal/loomrecipe#TestRouting_ProgressAt`
- `docs/overview.md`

**Create:**
- `plan:internal/loomrecipe#TestRecipe_DriftVerifyPrecedesCodeReview` -> `func TestRecipe_DriftVerifyPrecedesCodeReview(t *testing.T)`

**Intent:** Drift-Verify runs before the Code-Review perch in every loom recipe, as darn's order already does (D9).

`tapestry-recipe.yaml` moves the Drift-Verify row to follow Parent-Align: Parent-Align's `on_done` is Drift-Verify, Drift-Verify's `on_done` is Code-Bouncer, and Code-Bouncer's `on_done` is Describe; the row's place in the file follows its new order, and the header's Drift-Verify paragraph says it compares the code after Parent-Align, before the perch, so the review sees Drift-Verify's fixes, and that Finalize's reroute to Parent-Align passes Drift-Verify again, which compares whatever HEAD it meets against the same baseline (internal/driftengine/doc.go). The header also says a run in flight parked on Drift-Verify after its review runs the perch once more, and one inside the perch exits to Describe without Drift-Verify, which the orch moves back with `lym loom goto --to Drift-Verify`.

`TestRecipe_DriftVerifyPrecedesCodeReview` (tier0) walks `Routing("tapestry", …)`: Parent-Align → Drift-Verify → Code-Bouncer → Describe on `on_done`, and Finalize's reroute target Parent-Align reaches Drift-Verify by `on_done`. `wantProducerTable`, `TestRowNames_FollowEachRecipesRowOrder` and `TestRouting_ProgressAt` follow the new row order. The package doc's tapestry tail and `docs/overview.md`'s tail read `Parent-Align → Drift-Verify → Code-Review → Describe → PR-Review → Finalize → Friction-Reflect`.

**ImpactSummary:** Three edges and one row position in the tapestry recipe; fixtures pinning row order follow, and no Go producer changes.

# Card 10 — drift-verify-before-review

**Edit:**
- `contracts/recipes/tapestry-recipe.yaml`
- `internal/loomrecipe/doc.go`
- `internal/loomrecipe#wantProducerTable`
- `internal/loomrecipe#TestRowNames_FollowEachRecipesRowOrder`
- `internal/loomrecipe#TestRouting_ProgressAt`
- `internal/loomrecipe#TestEntryNumbers`
- `docs/overview.md`

**Intent:** Drift-Verify runs before the Code-Review perch in every loom recipe, as darn's order already does (D9).

`tapestry-recipe.yaml` moves the Drift-Verify row to follow Parent-Align: Parent-Align's `on_done` is Drift-Verify, Drift-Verify's `on_done` is Code-Bouncer, and Code-Bouncer's `on_done` is Describe; the row's place in the file follows its new order, and the header's Drift-Verify paragraph says it compares the code after Parent-Align, before the perch, so the review sees Drift-Verify's fixes, and that Finalize's reroute to Parent-Align passes Drift-Verify again, which compares whatever HEAD it meets against the same baseline (internal/driftengine/doc.go). The header also says a run in flight parked on Drift-Verify after its review runs the perch once more, and one inside the perch exits to Describe without Drift-Verify, which the orch moves back with `lym loom goto --to Drift-Verify`.

The tail subtest of `TestRouting_ProgressAt` asserts the new edges row by row: Parent-Align's `on_done` is Drift-Verify, Drift-Verify's is Code-Bouncer, Code-Bouncer's is Describe, and Finalize reroutes to Parent-Align, from which Drift-Verify follows by `on_done`; its title names the new order. `wantProducerTable` and `TestRowNames_FollowEachRecipesRowOrder` pin the file order. `TestEntryNumbers`' tapestry map gives Drift-Verify the number Code-Bouncer held and the Code-Bouncer, Code-Burler-Review and Code-Burler-Fix entries the one after it. The package doc's tapestry tail and `docs/overview.md`'s tail read `Parent-Align → Drift-Verify → Code-Review → Describe → PR-Review → Finalize → Friction-Reflect`.

**ImpactSummary:** Three edges and one row position in the tapestry recipe; fixtures pinning row order follow, and no Go producer changes.

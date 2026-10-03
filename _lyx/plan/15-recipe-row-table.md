# Card 15 — recipe-row-table

**Edit:**
- `internal/loomrecipe/shape_test.go`
- `internal/loomrecipe/recipe_test.go`
- `internal/loomrecipe/sequence_test.go`
- `internal/loomrecipe/coverage_guard_test.go`
- `internal/loomrecipe/interruptpolicy_meta_test.go`
- `internal/loomrecipe/names.go`

**Intent:** Pin the loom recipe's row list once (D9, #327 §1).
Delete `TestNew_ProducerTable` and `TestNew_ProducerTableOrderUnchangedByWiring`, strict subsets of `TestNew_ShapeMatchesRecipe`.
`wantProducerTable` keeps, per row in recipe order, the row name, the concrete producer type and the backing engine name (the column `coverage_guard_test.go`'s `loomRowEngines` carries today, which `shedengine.ProducerDef` cannot derive);
its literal `onStuck`/`onDone`/`segment`/`maxBounces` columns are dropped.
`TestNew_ShapeMatchesRecipe` keeps asserting row order and concrete type against it, and gains the perch-shape property in their place: every Bouncer's `OnStuck` names a Burler whose `OnStuck` names it back, with equal `Segment` and `MaxBounces`, and no row is an orphan (unreachable through `OnDone`/`OnStuck` from the first row).
`coverage_guard_test.go` reads its row-to-engine mapping from the one table, keeping all three directions and its every-direction-fails self-test; it is restructured, never weakened.
`interruptpolicy_meta_test.go` and `recipe_test.go`, the other readers of `loomRowEngines`, read the same mapping from the one table, and `interruptpolicy_meta_test.go`'s file comment, which explains its location by that variable, names the table instead.
`RecipeEngines`' doc comment in `names.go`, which contrasts itself with `loomRowEngines`, names the one row table instead;
that comment is the card's only production edit and changes no behaviour.
`sequence_test.go` derives `wantSequenceOrder` from the same table's order and types instead of carrying its own list: Done for an ordinary row, Bouncer Stuck, Burler Stuck, Bouncer Done for a review segment, and Stuck at Publish, where the sequence stops;
its doc comment explaining each entry's shape stays.
Prove it by swapping two `OnDone` targets in `contracts/recipes/loom-recipe.yaml`, watching a test fail, and restoring the file;
name the proof and the deleted tests in the commit body.
What this lets through, bounded per D9: a pair's `max_bounces` value or a consistently renamed segment label changes unnoticed, except Discussion-Bouncer's budget, which `resume_test.go` drives;
`OnDone` order through Publish stays guarded by `sequence_test.go`, and the PR-Gate and PR-Rework edges by `TestRecipe_PublishRoutesThroughPRGate`.

**ImpactSummary:** `internal/loomrecipe` tests plus one production doc comment; three copies of the row list become one table plus a perch-shape property.

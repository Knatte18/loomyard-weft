# Card 18 — the amendment_target recipe key

**Edit:**
- `internal/shedbuild#Recipe`
- `internal/shedbuild#TestParse_ValueAndErrorCases`
- `contracts/recipes/loom-recipe.yaml`
- `contracts/specs/shed-recipe-spec.md`
- `internal/shedbuild/doc.go`
- `internal/loomrecipe/doc.go`

**Create:**
- `internal/loomrecipe#AmendmentTarget`
- `internal/loomrecipe#TestAmendmentTarget`

**Intent:**
D8: a new optional top-level recipe key, `amendment_target`, names the row a plan amendment routes to; loom's recipe sets it to `Plan-Burler-Review`.

`shedbuild.Recipe` gains `AmendmentTarget string` (`amendment_target`), graph metadata `Build` never consumes, like `Entry` and `Terminals`.
`Parse` decodes with `KnownFields(true)`, so the field is what lets the key through; Parse does not check that it names a row, since routing is `shedengine`'s and `shedcheck`'s (the spec's validation split).
`contracts/recipes/loom-recipe.yaml` sets `amendment_target: Plan-Burler-Review` beside `entry`, with a comment that the arbiter's prompt names it once the plan is approved; the darn and batten recipes set none.
`loomrecipe.AmendmentTarget(recipe)` takes the recipe name as `Rows` does, answering for `shedrun.RecipeLoom`, `RecipeDarn` and `RecipeBatten`: it parses that embedded recipe and returns its key, empty when the recipe sets none; any other name is refused as `Rows` refuses it, and a parse failure is a `loomrecipe:`-prefixed error.
`shed-recipe-spec.md`'s document-shape paragraph and `shedbuild/doc.go` name the optional key; `loomrecipe/doc.go` names `AmendmentTarget`.

Tests:
`TestParse_ValueAndErrorCases` gains a row decoding a recipe with `amendment_target` into the field, and one without it decoding to empty.
`TestAmendmentTarget` asserts loom's answer is `Plan-Burler-Review`, darn's and batten's are empty, and an unknown name is an error.

**ImpactSummary:** one new optional field on the decoded recipe; every recipe without the key decodes as before, and only loom's recipe gains the key.

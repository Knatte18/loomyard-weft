# Card 32 — loomcli-beam-recipe

**Create:**
- `internal/loomcli#beamDescribeDeps`
- `internal/loomcli#TestWire_BeamRecipe`

**Edit:**
- `internal/loomcli#loomCLI.recipeRouting`
- `internal/loomcli#loomCLI.newShed`
- `internal/loomcli#loomCLI.verifySource`
- `internal/loomcli#tacklerFor`
- `internal/loomcli#loomCLI.wire`
- `internal/loomcli#landingDeps`
- `internal/loomcli#codeReviewOff`
- `internal/loomcli#fixerPlanDir`
- `internal/loomcli#fixerTestFileMatcher`
- `internal/loomcli#TestRecipeRouting`

**Extends:**
- `internal/loomcli/doc.go`

**Uses:**
- `internal/loomrecipe#NewBeam`
- `internal/loomrecipe#BeamRouting`
- `internal/shedrun#RecipeBeam`
- `internal/boardengine#OpenHub`

**Intent:**
Loom's bootstrap and verbs run a seed whose recipe is `beam`, the landing run batten seeds in a beam's pair (card 33).
`recipeRouting`, `newShed` and `tacklerFor` select `BeamRouting`, `NewBeam` and a tackler recipe line for `beam` ("beam: a beam's landing run (its description, then Finalize into its parent)").
`verifySource` gives a beam run the hub's gate-call source with no plan and no `darn.yaml` tree scope, so Finalize runs the `finalize` gate key over the whole merged tree.
`landingDeps` builds a beam run's Finalize with `LargeMergeLands` set, `ConflictReroutes` false, no failure tracer and a nil `RerouteReviewed`, so a large merge runs the gate and a spent or over-cap fixer halts for the orch (card 29); `codeReviewOff`, `fixerPlanDir` and `fixerTestFileMatcher` read a beam run as a darn run reads, with no plan and any-language test files.
`wire` fills `Env.BeamDescribe` through `beamDescribeDeps`: the project's title and brief from the hub board's entry for the run's slug (`boardengine.OpenHub`), the beam's own commit subjects since its merge base with the parent branch, the description path under the landing dir, and the description commit loom's Describe uses.
`TestRecipeRouting` gains a `beam` row whose routing has the two rows and no reroute edge; `TestWire_BeamRecipe`, like `TestWire_DarnRecipe`, wires a beam-seeded run over a hub and asserts the shed's rows and a set `BeamDescribe` seam, and asserts on the `landingDeps` result for the `beam` recipe, as the `codeReviewOff` and `fixerPlanDir` reads are judged, that `LargeMergeLands` is set, `ConflictReroutes` is false and `RerouteReviewed` is nil, so a spent or over-cap fixer halts.

**ImpactSummary:** Loom's recipe switches gain a third case; loom and darn runs take their existing branches.

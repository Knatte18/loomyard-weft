# Card 5 — sampler-registry-entry

**Create:**
- `contracts/recipes/sampler-recipe.yaml`
- `contracts/recipes#SamplerRecipe`
- `internal/shedrecipe#samplerSessionEntry`
- `internal/shedrecipe#TestSamplerSessionEntry`

**Edit:**
- `internal/shedrecipe#registry`
- `internal/shedrecipe#LoomBuildEngines`
- `internal/shedrecipe#Env`
- `internal/shedrecipe#newTestEnv`
- `internal/shedrecipe#minimalEntries`
- `internal/shedrecipe#TestNames`
- `internal/shedbuild#checkLoomContract`
- `internal/shedbuild#TestBuild_LoomContract`
- `internal/testkit/envkit#FullEnv`
- `internal/testkit/envkit#nilLegal`
- `internal/testkit/envkit#TestFullEnv_FillsEverySeam`
- `internal/testkit/envkit#TestNilSeams_ReportsExactPaths`
- `internal/testkit/envkit#TestNilSeams_SkipsNilLegalPaths`
- `internal/testkit/envkit#TestFullEnv_PathFieldsAbsoluteUnderOneRoot`
- `internal/loomcli#intentionallyNil`
- `internal/loomcli#TestWire_EverySeamFilled`
- `internal/loomcli#TestWire_BeamRecipe`
- `internal/loomcli#TestWire_DarnRecipe`
- `internal/battencli#intentionallyNil`
- `internal/battencli#TestWire_EverySeamFilled`
- `internal/loomrecipe#RecipeEngines`
- `internal/loomrecipe#TestRecipeEngines_ReportsExactlyLoomsOwnEngineSet`

**Extends:**
- `internal/shedrecipe/doc.go`
- `contracts/recipes/recipes.go`

**Uses:**
- `internal/shedrecipe#prReviewEntry`
- `internal/testkit/envkit#TestSeamMarkers_ReflectiveTestsAreMarked`
- `internal/shedrecipe#darnWriteEntry`
- `contracts/recipes/darn-recipe.yaml`

**Intent:**
D1 and `PATTERN-shed-recipe-registry`: `SamplerSession` is one new entry of the one `registry` map, built by `samplerSessionEntry`, and joins `LoomBuildEngines`, so a recipe whose build row runs it passes the loom contract's build-row clause.
`Env` gains `Sampler loomshed.SamplerDeps`, a whole-struct passthrough read by the SamplerSession entry only, documented as `Darn` is.
`samplerSessionEntry` refuses every config key with a way forward naming the key's removal, as `prReviewEntry` does, refuses each nil seam of `Env.Sampler` but `Watch` by field name through `requireSeam` and each empty record path through `requireAbsRoot`, and returns `loomshed.NewSamplerSession(row.Name, env.Sampler)`.
`checkLoomContract`'s engine-choice text becomes `Implementer, DarnWrite, BeamAwait or SamplerSession`, and `TestBuild_LoomContract`'s two expected way-forward strings follow it; `contracts/specs/shed-recipe-spec.md` names the set as `shedrecipe.LoomBuildEngines` and needs no change.
`TestSamplerSessionEntry` holds only what `TestEveryEntry_RejectsUnknownConfigKey` does not already reach through the new `minimalEntries` row (a built producer and the refused `bogus_key`): a nil `Sampler.Stamp` and a nil `Sampler.Sessions` are each refused naming the field, an empty `Sampler.RecordPath` is refused naming it, and a nil `Sampler.Watch` builds.
`minimalEntries` gains a `SamplerSession` row with empty config, so `TestEveryEntry_RejectsUnknownConfigKey` covers the entry, `TestNames`' `ShipsExpectedEntries` subtest gains `"SamplerSession"` in its sorted expected list, between `"RecordWrite"` and `"SeedChild"`, and `newTestEnv` and `envkit.FullEnv` fill `Sampler` with absolute paths under their one root and no-op seams; `nilLegal` gains `Sampler.Watch`.
Every test marked as reflecting over `shedrecipe#Env` gains the marker line `//lym:seam internal/loomshed#SamplerDeps`: the four envkit tests, the three loomcli wire tests and battencli's `TestWire_EverySeamFilled`.
`wire` fills no sampler seam for a tapestry seed and batten runs no sampler row, so loomcli's `intentionallyNil` lists each func seam of `Sampler` but `Watch` as "sampler-only seam, filled by wire for a sampler seed", and battencli's as "sampler-only seam, batten runs no sampler row".
`shedrecipe/doc.go` gains a `# The SamplerSession row` section saying it takes no config key, reads `Env.Sampler` and is a loom build engine.
So that `internal/shedrecipe#TestCoverageGuard` holds at this card, the card also writes the sampler recipe file and reaches it from `RecipeEngines`; the recipe's name, vocabulary and accessors come in card 6.
`sampler-recipe.yaml` opens with a comment block in the style of `darn-recipe.yaml`'s, declares `version: 1`, `entry: Preflight`, `contract: loom`, `verify_source: tree_scope` and `terminals: [Friction-Reflect]` (`PATTERN-loom-contract`), and its rows are, in order: `Preflight` (engine Preflight, on_done Record-Write); `Record-Write` (RecordWrite, on_done Sampler-Session); `Sampler-Session` (SamplerSession, no config, on_done Code-Bouncer); `Code-Bouncer`, `Code-Burler-Review` and `Code-Burler-Fix` copied from `darn-recipe.yaml` with the same segment, edges, config and answer key, each also carrying `entry_gate: sampler.code_review` (`PATTERN-shed-recipe-registry`: the perch is left out by its gate, never by a producer switch); `Describe` (Describe, no entry_gate, its `description` gate of 3 attempts, on_done PR-Review), since the agent writes no description and Finalize parses one; `PR-Review` (PRReview, on_done Finalize); `Finalize` (Finalize, on_reroute Code-Bouncer, on_done Friction-Reflect); `Friction-Reflect` (FrictionReflect, on_done "").
It has no Drift rows and no Parent-Align row; the comment says why (the work is prose, board entries and trials, and Finalize merges the parent under the parent lock), that the build row runs no verify gate and Finalize's gate checks any code the run lands, and that with `code_review` false the three perch rows are recorded Skipped and Describe still writes the description.
`recipes.go` embeds it as `SamplerRecipe`, and `RecipeEngines` parses it with the other loom recipes, its doc naming `recipes.SamplerRecipe` among its sources and its "All three recipes are the loom module's" sentence becoming four, so the coverage guard sees `SamplerSession`; `TestRecipeEngines_ReportsExactlyLoomsOwnEngineSet` gains `SamplerSession`.

**ImpactSummary:** registers one engine and one Env field, and embeds the sampler recipe file for the coverage guard's engine set; every Env-reflecting seam test and fixture builder in shedrecipe, envkit, loomcli and battencli is updated, and the registry name pin gains the engine; `checkLoomContract`'s engine-choice text changes, and `TestBuild_LoomContract`'s expected strings with it; widening `LoomBuildEngines` also reaches its other consumer, `loomrecipe.verifyGatedRows`, which card 6 narrows.

**Commit:** 5: shedrecipe: register the SamplerSession build engine and embed the sampler recipe

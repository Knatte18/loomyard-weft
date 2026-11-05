# Card 23 — raddle-recipe

**Create:**
- `contracts/recipes/raddle-recipe.yaml`
- `plan:contracts/recipes#RaddleRecipe` -> `var RaddleRecipe []byte`
- `plan:internal/loomshed#NameRaddleWrite` -> `const NameRaddleWrite = "Raddle-Write"`
- `plan:internal/loomshed#NameRaddleFile` -> `const NameRaddleFile = "Raddle-File"`
- `plan:internal/shedrecipe#RaddleWriteEnv` -> `type RaddleWriteEnv struct`
- `plan:internal/shedrecipe#raddleWriteEntry` -> `func raddleWriteEntry(row RowIdentity, cfg Config, env Env) (shedengine.ShedProducer, error)`
- `plan:internal/shedrecipe#raddleFileEntry` -> `func raddleFileEntry(row RowIdentity, cfg Config, env Env) (shedengine.ShedProducer, error)`
- `plan:internal/shedrecipe#TestRaddleEntries` -> `func TestRaddleEntries(t *testing.T)`
- `plan:internal/loomrecipe#TestRaddleRecipe` -> `func TestRaddleRecipe(t *testing.T)`
- `plan:internal/loomcli#raddleWriteEnv` -> `func raddleWriteEnv(location *lymcwd.Location, cfg loomengine.Config, stencilsDir, specsDir string, reg modelspec.Registry, slug string, setting func(module, key string) (any, error)) shedrecipe.RaddleWriteEnv`
- `plan:internal/loomcli#raddleFileDeps` -> `func raddleFileDeps(location *lymcwd.Location, slug string) loomshed.RaddleFileDeps`
- `plan:internal/loomcli#createBeamFromParent` -> `func createBeamFromParent(location *lymcwd.Location, project string, run func(dir string, args ...string) error) error`
- `plan:internal/loomcli#TestCreateBeamFromParent` -> `func TestCreateBeamFromParent(t *testing.T)`
- `plan:internal/battencli#TestChildRecipeSupported` -> `func TestChildRecipeSupported(t *testing.T)`

**Edit:**
- `internal/shedrun#RecipeNames`
- `internal/shedrun#TestRecipeNames`
- `internal/shedrun#TestValidateRecipe`
- `internal/loomrecipe#embeddedRecipes`
- `internal/loomrecipe#RecipeEngines`
- `internal/loomrecipe#RowNames`
- `internal/loomrecipe#Rows`
- `internal/loomrecipe#AmendmentTarget`
- `internal/loomrecipe#TestRecipeEngines_ReportsExactlyLoomsOwnEngineSet`
- `internal/loomrecipe#TestInterruptPolicies_MatchAssembledRows`
- `internal/loomrecipe#wantProducerTable`
- `internal/loomrecipe#testEnv`
- `internal/loomshed#InterruptPolicies`
- `internal/shedrecipe#registry`
- `internal/shedrecipe#Env`
- `internal/shedrecipe#resolvePlainGateClosure`
- `internal/shedrecipe#TestNames`
- `internal/shedrecipe#TestResolveGateSpec_UnknownNameListsTheLegalValues`
- `internal/shedcli#recipes`
- `internal/shedcli#TestRecipes_KeySet`
- `internal/shedcli#TestRecipes_Entries`
- `internal/battencli#childRecipeSupported`
- `internal/battencli#unsupportedChildRecipeRefusal`
- `internal/battencli#childSeedParams`
- `internal/runprofile/template.yaml`
- `internal/runprofile_test#TestLoad`
- `internal/loomcli#loomCLI.wire`
- `internal/loomcli#finishesAsSketch`
- `internal/loomcli#arbiterRecipeDescription`
- `internal/loomcli#TestRecipeKeyedWiring`
- `internal/loomcli/doc.go`
- `internal/shedrun/doc.go`
- `internal/loomrecipe/doc.go`
- `internal/shedcli/doc.go`
- `internal/runprofile/doc.go`

**Extends:**
- `internal/shedrecipe/doc.go`

**Uses:**
- `internal/shedbuild#checkLoomContract`
- `internal/shedrecipe#LoomRecordEngines`
- `internal/shedrecipe#discussionWriteEntry`
- `internal/shedrecipe#recordWriteEntry`
- `internal/shedadapters#NewMultiLLMProducerSourced`
- `internal/loomshed#NewRaddleWrite`
- `internal/loomshed#NewRaddleFile`
- `internal/loomshed#NewRaddleGate`
- `internal/loomshed#StoreColdProposal`
- `internal/loomengine#RaddleTable`
- `internal/loomcli#raddleCheckFor`
- `internal/loomcli#raddleBoardContext`
- `internal/boardengine#Board.FileRaddle`
- `internal/boardengine#Board.ReleaseTasks`
- `internal/boardengine#Board.EndRaddleEntry`
- `internal/hubgeom#ParentOf`
- `internal/orchcli#NotifyParent`
- `internal/loomcli#TestWire_EverySeamFilled`
- `internal/battencli#TestWire_EverySeamFilled`
- `internal/loomcli#TestWire_BeamRecipe`
- `internal/loomcli#TestWire_DarnRecipe`
- `internal/loomcli#TestWire_SamplerRecipe`
- `internal/testkit/envkit#TestFullEnv_FillsEverySeam`
- `internal/testkit/envkit#TestFullEnv_PathFieldsAbsoluteUnderOneRoot`
- `internal/testkit/envkit#TestNilSeams_ReportsExactPaths`
- `internal/testkit/envkit#TestNilSeams_SkipsNilLegalPaths`
- `internal/testkit/envkit#TestSeamMarkers_ReflectiveTestsAreMarked`
- `internal/shedrecipe_test#TestCoverageGuard`
- `internal/shedrun#RecipeRaddle`
- `internal/loomshed/doc.go`

**Intent:** D1: the `raddle` recipe, its two engines' registry rows and its wiring, in one card because `TestWire_EverySeamFilled` checks every `Env` seam a new field adds is filled by the wiring in the same build.
`raddle-recipe.yaml` declares `contract: loom` in its record-only form, `entry: Record-Write`, terminal `Raddle-File`, and three rows: `Record-Write` (engine `RecordWrite`, `on_done: Raddle-Write`); `Raddle-Write` (engine `RaddleWrite`, `segment: Raddle`, `on_done: Raddle-File`, `gates: [{name: raddle, attempts: 3}]`, no `on_stuck`, so the third failing gate attempt halts the run for its parent like Discussion-Write's); `Raddle-File` (engine `RaddleFile`, `segment: Raddle`, `on_stuck: Raddle-Write`, `max_bounces: 3`, `on_done: ""`), with a header comment giving the reasons as `motif-recipe.yaml`'s does.
Record-Write stays the one record row (`LoomRecordEngines` unchanged), and neither new engine is a build or Finalize engine, so `checkLoomContract` accepts the recipe as record-only with no change; the shared segment lets Raddle-File's Stuck return to Raddle-Write, and `max_bounces: 3` stays above Raddle-File's own two-return halt, so the producer, not the budget, halts the third failure.
`recipes.go` embeds it as `RaddleRecipe`; `shedrun.RecipeRaddle`, the constant card 13 declared, joins `RecipeNames` (sorted, between pawl and sampler) and so `ValidateRecipe`; `loomrecipe`'s `embeddedRecipes` and `RecipeEngines` take it; `shedcli`'s `recipes` arms it like motif (`loomcli.ArmAt`, the run, step, status, pause and goto verbs), its map key `shedrun.RecipeRaddle`, never the literal `"raddle"`; batten's `childRecipeSupported` admits it, `unsupportedChildRecipeRefusal` lists it and `childSeedParams`' comment counts it, so batten runs it like motif: in a worktree of its own forked from the parent, with a pane, a board status and notices, landing no code.
Every place this card writes the recipe's name in production code, `embeddedRecipes`, `RecipeEngines`, `childRecipeSupported` and `unsupportedChildRecipeRefusal` as well as `recipes`, takes `shedrun.RecipeRaddle` (`PATTERN-named-constants`); a test file may spell the YAML key as the profile template does.
`loomshed.InterruptPolicies` gives `Raddle-Write` the policy `Discussion-Write` has and `Raddle-File` the re-invoke policy, since every step of it is idempotent; `TestInterruptPolicies_MatchAssembledRows` takes the raddle recipe into its loop and its failure message, which lists the recipes, names raddle.
`wantProducerTable` gains the Raddle-Write and Raddle-File rows with their engines and producer types; the doc comments of `RowNames`, `Rows`, `RecipeEngines` and `AmendmentTarget`, which list the recipes, name raddle.
`shedrecipe.Env` gains `RaddleWrite RaddleWriteEnv` (the `loomshed.RaddleWriteDeps`, the table source `func(interactive bool, returnNote string) (seatengine.Table, error)` and the `loomshed.RaddleCheck`) and `RaddleFile loomshed.RaddleFileDeps`, whole-struct passthroughs like `RecordWrite`.
`raddleWriteEntry`, built like `discussionWriteEntry`, rejects unknown keys but `gates`, requires its seams, builds the inner `NewMultiLLMProducerSourced` over a table source that reads the recorded mode and return from the raddle state each call and sets the resolved gate, and wraps it in `loomshed.NewRaddleWrite`; `raddleFileEntry` rejects every key, requires every seam and returns `loomshed.NewRaddleFile`.
`resolvePlainGateClosure` gains the `raddle` case, `loomshed.NewRaddleGate(env.RaddleWrite.Check, env.RaddleWrite.Deps.StoreCold)`, and the unknown-name error lists it.
`registry` gains both engines, which `TestNames` pins and `TestCoverageGuard` finds reached through `RecipeEngines`.
`loomCLI.wire` fills both fields for every recipe, as it fills the others: `raddleWriteEnv` (the table through `loomengine.RaddleTable`, the check through `raddleCheckFor`, `AutoApprove` through the run's setting lookup of `loom.raddle_auto_approve`, `FreshEntry` from the run's status history, `StoreCold` through `loomshed.StoreColdProposal` over the raddle paths, and the raddle directory commit) and `raddleFileDeps` (the board writes through `boardengine.OpenHub`, `StoreDone` through `ReadRaddleStore`, `Entry` reading the entry's `created` stamp, labels and own `project` field from the board, the level D9 files at, `Notify` through `orchcli.NotifyParent`, `EndEntry` through `EndRaddleEntry`, and `CreateBeam` through `createBeamFromParent`).
`createBeamFromParent` resolves the parent worktree of the run's slug through `hubgeom.ParentOf` (`PATTERN-pair-parent`), the prime when it names none, and runs `lym beam create <project>` there as a child process it waits on (`PATTERN-reap-children`), logging the spawn (`PATTERN-spawn-observability`); `beamcli`'s create already skips each step whose result exists, so a retry finishes it.
`finishesAsSketch` is false for a recipe with a `RaddleFile` row as well as for one with a Finalize row, so a finished raddle run never makes its entry a Sketch (D10), chosen by the recipe's rows.
`loomcli/doc.go` rewords its sentence "A finished run of a recipe with no Finalize row, a motif run, makes its entry a Sketch" so it no longer covers a raddle run, and says that a recipe with a `RaddleFile` row ends its entry done in its own Raddle-File instead of as a Sketch.
The other doc files that list the recipes gain raddle: `shedrun/doc.go`'s list of the recipe names a seed may hold; `loomrecipe/doc.go`'s three recipe lists (the `New` and `Routing` line, the `Rows` and `RowNames` line, the `AmendmentTarget` line); `shedcli/doc.go`'s list of the recipe names its table maps; `runprofile/doc.go`'s recipe list and its defaults sentence (`solid` for tapestry and motif, `light` for darn and pawl), where raddle takes `light`.
`arbiterRecipeDescription` describes a recipe with a `RaddleFile` row, chosen by its rows before the no-Finalize case, as a run that splits a Sketch's stored decision record into board entries the operator approves, landing no code; its doc comment names that case, and `TestRecipeKeyedWiring`'s raddle row asserts the description.
`runprofile/template.yaml` gains `raddle: light` under `defaults` and a `raddle` block in each profile, `loom: {raddle_auto_approve: false}`, its header naming the recipe; `TestLoad`'s default map gains it.
`TestRaddleRecipe` (tier0, `testEnv` gaining the raddle seams) builds the recipe and asserts the rows in order Record-Write, Raddle-Write, Raddle-File, Raddle-File's `on_stuck` and `max_bounces`, the `raddle` gate with 3 attempts, and that the loader accepts it under the record-only form.
`TestRaddleEntries` (tier0) has rows: each entry built from a full env; each refusing an unset seam; `raddleFileEntry` refusing an unknown key.
`TestCreateBeamFromParent` (tier0) asserts the child runs `beam create <project>` in the parent worktree, and a failed run is returned.
`TestChildRecipeSupported` (tier0) has rows: `raddle` admitted beside the other loom recipes, and an unknown recipe refused by `unsupportedChildRecipeRefusal` with a message listing `raddle`.
`TestRecipeKeyedWiring` gains the raddle row (not finishing as a Sketch, and the arbiter description above); `TestRecipeNames`, `TestValidateRecipe`, `TestRecipes_KeySet`, `TestRecipes_Entries` and `TestRecipeEngines_ReportsExactlyLoomsOwnEngineSet` gain the recipe and its engines; `TestResolveGateSpec_UnknownNameListsTheLegalValues` the gate name.

**ImpactSummary:** One new recipe threaded through the closed recipe lists, the engine registry, the gate vocabulary, batten's child set and loom's wiring; every existing recipe's rows and keys are unchanged.

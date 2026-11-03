# Card 6 — loom-recipes-motif-arras

**Create:**
- `//contracts/recipes/motif-recipe.yaml`
- `//contracts/recipes/arras-recipe.yaml`
- `plan:contracts/recipes#MotifRecipe` -> `var MotifRecipe []byte`
- `plan:contracts/recipes#ArrasRecipe` -> `var ArrasRecipe []byte`
- `plan:internal/shedrun#RecipeMotif` -> `const RecipeMotif = "motif"`
- `plan:internal/shedrun#RecipeArras` -> `const RecipeArras = "arras"`
- `plan:internal/loomshed#NameMotif` -> `const NameMotif = "Motif"`
- `plan:internal/loomrecipe#embeddedRecipes` -> `var embeddedRecipes map[string][]byte`
- `plan:internal/loomrecipe#TestBlockRecipes` -> `func TestBlockRecipes(t *testing.T)`

**Edit:**
- `internal/loomrecipe#embeddedRecipeSource`
- `internal/loomrecipe#parseEmbeddedRecipe`
- `internal/loomrecipe#RecipeEngines`
- `internal/loomrecipe#RecipeRow`
- `internal/loomrecipe#Rows`
- `internal/loomrecipe#RowNames`
- `internal/loomrecipe#AmendmentTarget`
- `internal/loomrecipe#Routing`
- `internal/loomrecipe#reviewSegments`
- `internal/loomrecipe#BouncerRunSubdir`
- `internal/loomrecipe#IsFinalizeRow`
- `internal/loomrecipe/doc.go`
- `internal/loomrecipe#TestRowNames_FollowEachRecipesRowOrder`
- `internal/loomrecipe#TestEntryNumbers`
- `internal/loomrecipe#TestRouting_ProgressAt`
- `internal/loomrecipe#TestRecipeEngines_ReportsExactlyLoomsOwnEngineSet`
- `internal/loomrecipe#TestRecipe_StructuralCheckHasNoFindings`
- `internal/loomrecipe#TestReviewBudget_AppliedByNewAndRouting`
- `internal/loomrecipe#TestRecipeKeys`
- `internal/loomrecipe#TestInterruptPolicies_MatchAssembledRows`
- `internal/loomshed#InterruptPolicies`
- `internal/shedrecipe_test#coverageGuardAllowedUnreachableEngines`

**Uses:**
- `plan:internal/shedbuild#Expand`
- `plan:internal/shedbuild#RoutingFrom`
- `internal/loomrecipe#testEnv`
- `internal/loomshed/doc.go`
- `//contracts/recipes/tapestry-recipe.yaml`

**Intent:** Ship the two recipes and make every loom accessor read the expanded recipe through one table (D9, D10, D11, D7).

Recipes, embedded in `contracts/recipes/recipes.go` beside the others:
- `motif-recipe.yaml` sets only `version`, `entry: Discussion-Write` and `terminals: [Discussion-Bouncer]` at top level, and holds tapestry's Discussion-Write, Discussion-Bouncer, Discussion-Burler-Review and Discussion-Burler-Fix rows copied verbatim (names, engines, whole config, segment, edges), bar Discussion-Bouncer's `on_done`, which is empty so the block rewrites it to its exit.
  Its header says motif is the recipe that makes a sketch, the decision record, from a task, that it is a block source only, and that `TestBlockRecipes` holds its rows equal to tapestry's.
- `arras-recipe.yaml` sets tapestry's top-level keys (`version`, `entry: Preflight`, `amendment_target`, `contract: loom`, `verify_source: plan`, `terminals`) plus `profile_recipe: tapestry`, then Preflight with `on_done: Motif`, the block row `name: Motif`, `block: motif`, `block_gate: always`, `on_done: Drift-Baseline`, then tapestry's rows from Drift-Baseline to Friction-Reflect copied verbatim. Its header names the block, says the `Motif` block produces the sketch, the decision record, and points at tapestry's header for the rows' routing.
Tapestry's recipe file is not touched.

Constants (PATTERN-named-constants): `shedrun.RecipeMotif` and `shedrun.RecipeArras` beside the other recipe names, `RecipeMotif` documented as the recipe that makes a sketch, the decision record, run only as a block, and neither added to `RecipeNames` on this card (card 7 adds `arras`); `loomshed.NameMotif` beside the other row names.

loomrecipe:
- `embeddedRecipes` is the one table from recipe name to embedded bytes: tapestry, darn, batten, pawl, sampler, motif and arras.
  `embeddedRecipeSource` reads it, an empty name still reading as tapestry, and is the resolver every expansion is told.
- `parseEmbeddedRecipe` parses, then expands through `shedbuild.Expand` with `embeddedRecipeSource`, so `New`, `Rows`, `RowNames`, `AmendmentTarget`, `IsLoom`, `HasEngine`, `EngineRow`, `GateFixerSetting`, `CodePerchGate`, `RecordEngine` and `VerifySource` all see the expanded rows.
  The batten, pawl and other recipes hold no block row, so their expansion returns them unchanged.
- `Routing` projects through `shedbuild.RoutingFrom` over the expanded recipe, and `reviewSegments` reads the expanded recipe, so arras's Discussion-Review segment takes its budget as tapestry's does.
- `RecipeEngines` collects engines over the expanded tapestry, darn, pawl, sampler and arras recipes, so `Block` joins the set.
- `BouncerRunSubdir` and `IsFinalizeRow` read the tapestry and darn recipes through `parseEmbeddedRecipe` instead of parsing the sources themselves; their file comments say so.
- `RecipeRow` gains `BlockPath string`, `SkipTo string` and `IsBlock bool` (the row's engine is `shedrecipe.EngineBlock`), and `Rows` fills them.
- `RowNames`', `Rows`' and `AmendmentTarget`'s docs list the recipes they serve, motif and arras included, and say the rows come in expanded order.
- `doc.go` describes motif as the recipe that makes a sketch, the decision record, run as a block, arras, the one table and that every accessor reads the expanded recipe.

The coverage guard's allowlist drops `Block`, which `RecipeEngines` now reaches.

`loomshed.InterruptPolicies` gains `NameMotif` mapped to `shedverbs.InterruptPolicyReinvoke`, since the Motif producer returns Done at once and re-invoking it repeats nothing.

Tests, all tier0:
- `TestBlockRecipes` (new):
  - motif's rows equal tapestry's four Discussion rows by `reflect.DeepEqual` on the parsed `shedbuild.Row`s, bar Discussion-Bouncer's empty `on_done`;
  - arras's expanded rows equal tapestry's rows plus the `Motif` row, config included, bar Preflight's `on_done` (`Motif`), the inlined rows' `BlockPath` (`Motif`), and `Motif`'s own fields (engine `Block`, `OnDone` Discussion-Write, `SkipTo` Drift-Baseline);
  - arras's top-level keys equal tapestry's, `ProfileRecipe` aside;
  - `New(shedrun.RecipeArras, …)` over `testEnv` builds, so arras passes the loom contract check on its expanded rows.
- `TestRowNames_FollowEachRecipesRowOrder` compares each recipe's rows with its expanded recipe and gains arras and motif rows, asserting the new `RecipeRow` fields.
- `TestEntryNumbers` gains arras: Preflight 1, Motif 2, Discussion-Write 3, the Discussion-Review rows 4, Drift-Baseline 5, on to Friction-Reflect 18.
- `TestRouting_ProgressAt` gains an arras subtest: steps read Preflight, Motif, Discussion-Write, Discussion-Review, then tapestry's later steps, and `ProgressAt("Drift-Baseline")` is step 5, the step a pass-over moves to from Motif's step 2.
- `TestRecipeEngines_ReportsExactlyLoomsOwnEngineSet` adds `Block` to its want set.
- `TestRecipe_StructuralCheckHasNoFindings` gains arras, expanded before `shedbuild.Build`.
- `TestReviewBudget_AppliedByNewAndRouting` gains arras: the Discussion-Review rows carry the `Discussion-Review` budget in `New` and `Routing`.
- `TestRecipeKeys` gains rows: `IsLoom` is true for arras and false for motif; `HasEngine(arras, "DiscussionWrite")` is true and `EngineRow(arras, "DiscussionWrite")` is `Discussion-Write`.
- `TestInterruptPolicies_MatchAssembledRows` adds `shedrun.RecipeArras` to the recipes it builds, and its doc comment and reverse-direction error message name arras beside tapestry, darn and sampler.

**ImpactSummary:** Two new embedded recipes, one name table in loomrecipe and one `loomshed.InterruptPolicies` entry for the Motif row, which `shedverbs`' envelopes read; every accessor now expands, which leaves the five existing recipes' rows as they were, so their callers in loomcli, battencli, reportengine and shedverbs see no change.

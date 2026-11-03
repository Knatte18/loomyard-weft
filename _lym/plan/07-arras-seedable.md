# Card 7 — arras-seedable

**Edit:**
- `internal/shedrun#RecipeNames`
- `internal/shedrun/doc.go`
- `internal/shedrun#TestRecipeNames`
- `internal/shedrun#TestValidateRecipe`
- `internal/shedcli#recipes`
- `internal/shedcli/doc.go`
- `internal/shedcli#TestRecipes_KeySet`
- `internal/battencli#childRecipeSupported`
- `internal/battencli#unsupportedChildRecipeRefusal`
- `internal/battencli#childSeedParams`
- `internal/battencli#TestWire_WriteSeedRefusesANonLoomChildBeforeTouchingTheWorktree`
- `internal/battencli#TestChildSeedFor_CarriesRecipeDriverAndParams`
- `internal/battenshed#ErrUnsupportedChildRecipe`
- `internal/boardcli#storeVerbs`
- `internal/loomcli#arbiterRecipeDescription`
- `internal/loomcli#TestRecipeKeyedWiring`
- `internal/loomcli#TestWire_DefaultConfig`
- `internal/reportengine#runRecipes`
- `internal/reportengine#recipePhaseTables`
- `internal/reportengine/doc.go`
- `internal/reportengine#TestCompute_SelectsRuns`
- `internal/reportengine#TestComputeTiming`
- `internal/gateslot/doc.go`
- `contracts/stencils/landing/landing-template-pr-review.md`
- `tools/tokencount#countedRecipes`
- `tools/tokencount#recipeList`
- `tools/tokencount/main.go`
- `tools/tokencount#TestRecentRuns`

**Uses:**
- `plan:internal/shedrun#RecipeArras`
- `plan:internal/shedrun#RecipeMotif`
- `plan:internal/loomshed#NameMotif`
- `internal/reportengine#loomPhases`
- `internal/battenshed/doc.go`
- `internal/boardcli/doc.go`
- `internal/loomcli#loomSegmentPerches`

**Intent:** Make arras a seedable recipe accepted and wired wherever tapestry is accepted by name (D10); motif stays out of every one of these sites (D9).

- `shedrun.RecipeNames` gains `RecipeArras` in sorted position (arras first); `ValidateRecipe` follows, so a board task's `recipe` field, `lym shed seed --recipe` and batten's `WriteSeed` accept `arras`; `doc.go` names the vocabulary with arras and says motif, the recipe that makes a sketch, the decision record, is a block source only.
- shedcli's `recipes` table gains an `arras` entry armed as tapestry's is (`loomcli.ArmAt`, the same verbs, `loomcli.BootstrapVerb`); `doc.go`'s recipe list gains it.
- battencli's `childRecipeSupported` admits `RecipeArras`, and `unsupportedChildRecipeRefusal` names it among the admitted recipes and the task types; its doc's "four" becomes "five".
  `childSeedParams` follows, giving arras the parent param tapestry gets, and its doc names arras among the recipes that declare one; `battenshed.ErrUnsupportedChildRecipe`'s doc names arras among the recipes `lym loom start` serves.
- The board's `recipe` field help text in `storeVerbs` lists arras beside darn and tapestry.
- loomcli's `arbiterRecipeDescription` names a recipe with a Plan-Write row by `recipeLabel(recipe)` instead of the literal tapestry, with "an" before a label opening with a vowel, so an arras run reads "This is an arras run, a task driven from discussion to landing" and a tapestry run reads as before.
- reportengine's `runRecipes` gains arras, and `recipePhaseTables` gives arras tapestry's phase table with the `Motif` row added to the discussion phase; `doc.go` and `runs.go`'s file comment name arras beside tapestry and darn among the selected recipes.
- gateslot's `doc.go` and the PR-Review stencil's leading comment list arras among the recipes that run PR-Review.
- tokencount's `countedRecipes` gains arras, so the default selection counts finished arras runs; `recipeList` renders a list of three or more as `a, b <conjunction> c`, and `main.go`'s package comment names arras beside tapestry and darn.
The board's `default_recipe` is untouched.

Tests, all tier0:
- `TestRecipeNames` and `TestRecipes_KeySet` (its literal want and the `//testtiming:keep` reason's list) gain arras.
- `TestValidateRecipe` gains rows: `arras` accepted, `motif` refused naming the available recipes.
- `TestWire_WriteSeedRefusesANonLoomChildBeforeTouchingTheWorktree`'s admitted loops gain arras, and the refusal's text check follows the new wording; `TestChildSeedFor_CarriesRecipeDriverAndParams` gains an arras row carrying `params.parent`.
- `TestRecipeKeyedWiring` gains an Arras row: plan source, a plan directory, and the description fragment "a task driven from discussion to landing" with the label arras.
- `TestWire_DefaultConfig` gains an arras pass of its budget block: over the same default config, `loomrecipe.Routing(arras, loomSegmentBudgets(cfg))` and `loomrecipe.New` over the wired env give the Discussion-Review rows the budget `loomSegmentPerches` maps that segment to, the discussion perch's keys, as for tapestry.
- `TestCompute_SelectsRuns` gains an arras run selected as a tapestry run is; `TestComputeTiming` gains an arras run whose Motif row counts under the discussion phase.
- `TestRecentRuns` gains a finished arras run counted by the default selection, and asserts the three-recipe sentence.

**ImpactSummary:** Additive entries in five name-keyed tables, one refusal text, one description label and the docs that list recipe names; no existing recipe's acceptance or wiring changes.

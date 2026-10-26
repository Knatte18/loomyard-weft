# Card 6 — recipe-named-pawl

**Rename:**
- `contracts/recipes/beam-recipe.yaml` -> `contracts/recipes/pawl-recipe.yaml`

**Create:**
- `plan:internal/shedrun#RecipePawl` -> `const RecipePawl = "pawl"`
- `plan:contracts/recipes#PawlRecipe` -> `var PawlRecipe []byte`
- `plan:internal/loomrecipe#NewPawl` -> `func NewPawl(env shedrecipe.Env, paths shedbuild.ShedPaths) (*shedengine.Shed, error)`
- `plan:internal/loomrecipe#PawlRouting` -> `func PawlRouting(budgets map[string]int) (shedengine.Routing, error)`
- `plan:internal/loomcli#pawlTacklerRecipe` -> `const pawlTacklerRecipe = "pawl: a beam's life (its orch and dobby, the wait for its landing, its merge, review, description and Finalize into its parent)"`

**Delete:**
- `internal/shedrun#RecipeBeam`
- `contracts/recipes#BeamRecipe`
- `internal/loomrecipe#NewBeam`
- `internal/loomrecipe#BeamRouting`
- `internal/loomcli#beamTacklerRecipe`

**Edit:**
- `internal/shedrun#RecipeNames`
- `internal/shedrun#decodeSeed`
- `internal/shedrun#TestRecipeNames`
- `internal/shedrun#TestValidateRecipe`
- `internal/shedrun#TestParseSeed`
- `internal/shedcli#recipes`
- `internal/shedcli#TestRecipes_KeySet`
- `internal/loomrecipe#RecipeEngines`
- `internal/loomrecipe#RowNames`
- `internal/loomrecipe#Rows`
- `internal/loomrecipe#AmendmentTarget`
- `internal/loomrecipe#parseEmbeddedRecipe`
- `internal/loomrecipe#TestNewBeam`
- `internal/loomrecipe#TestBeamRouting`
- `internal/loomrecipe#TestRowNames_FollowEachRecipesRowOrder`
- `internal/loomcli#arbiterRecipeDescription`
- `internal/loomcli#codeReviewOff`
- `internal/loomcli#fixerPlanDir`
- `internal/loomcli#fixerTestFileMatcher`
- `internal/loomcli#landingDeps`
- `internal/loomcli#loomCLI.armAt`
- `internal/loomcli#loomCLI.newShed`
- `internal/loomcli#loomCLI.recipeRouting`
- `internal/loomcli#loomCLI.wire`
- `internal/loomcli#tacklerFor`
- `internal/loomcli#TestRecipeRouting`
- `internal/loomcli#TestWire_BeamRecipe`
- `internal/battencli#boardRecipe`
- `internal/battencli#childRecipeSupported`
- `internal/battencli#unsupportedChildRecipeRefusal`
- `internal/battencli#TestBoardRecipe`
- `internal/battencli#TestWire_WriteSeedRefusesANonLoomChildBeforeTouchingTheWorktree`

**Intent:**
D8's rename: the `beam` recipe becomes `pawl`, still the two rows Beam-Describe and Finalize here; card 16 grows its rows.

- `git mv contracts/recipes/beam-recipe.yaml contracts/recipes/pawl-recipe.yaml` first, per the Rename mechanic, then the header comment names the pawl recipe; `recipes.go` embeds it as `PawlRecipe`, and `BeamRecipe` goes.
- `shedrun.RecipeBeam` becomes `shedrun.RecipePawl` with the value `pawl`, in `RecipeNames` and so in `ValidateRecipe`'s vocabulary; `beam` leaves the vocabulary.
- A seed recording the recipe `beam` reads as `pawl`: `decodeSeed` maps it on read, which `ReadSeed` and `ParseSeed` share, so a landing run in flight at the upgrade resumes at its row and `WriteSeed`'s agreement check compares the mapped recipe; a seed is never refused for the old name (`PATTERN-tolerant-records`).
- `loomrecipe.NewBeam` and `BeamRouting` become `NewPawl` and `PawlRouting` over `PawlRecipe`; `PawlRouting` takes the review budgets as `DarnRouting` does and checks and applies them over the recipe's review segments, none until card 16, and `loomCLI.recipeRouting` passes `reviewBudget()` for a pawl run; and `RecipeEngines`, `RowNames`, `Rows`, `AmendmentTarget` and `parseEmbeddedRecipe` name `RecipePawl`.
- Every `RecipeBeam` branch in `loomcli` and `battencli` names `RecipePawl` with its behavior unchanged here; `beamTacklerRecipe` becomes `pawlTacklerRecipe`, and `arbiterRecipeDescription` describes a pawl run as a beam's life driven through the loom engine.
  `boardRecipe` seeds `pawl` for a project entry, `childRecipeSupported` admits `pawl`, and `unsupportedChildRecipeRefusal` names it.
- `shedcli.recipes` keys the entry `pawl`.

Tests: `TestParseSeed` gains a row decoding `{"recipe":"beam"}` as `pawl`; `TestRecipeNames`, `TestValidateRecipe`, `TestRecipes_KeySet` and `TestRowNames_FollowEachRecipesRowOrder` name `pawl` in place of `beam`; `TestNewBeam`, `TestBeamRouting`, `TestRecipeRouting` and `TestWire_BeamRecipe` drive the pawl names.
`TestBoardRecipe` gains a row pinning `boardengine.ProjectRecipe` equal to `shedrun.RecipePawl` and asserts a project seeds `pawl`.

**ImpactSummary:** a vocabulary rename across shedrun, contracts/recipes, loomrecipe, loomcli, battencli and shedcli with every caller listed; the only behavior added is reading an old `beam` seed as `pawl`.

**Commit:** 6: The beam recipe is renamed pawl; a `beam` seed reads as `pawl`

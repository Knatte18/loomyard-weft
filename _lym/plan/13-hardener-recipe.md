# Card 13 — hardener-recipe

**Create:**
- `contracts/recipes/hardener-recipe.yaml`
- `plan:contracts/recipes#HardenerRecipe` -> `var HardenerRecipe []byte`
- `plan:internal/shedrun#RecipeHardener` -> `const RecipeHardener = "hardener"`
- `plan:internal/loomshed#NameHardenBouncer` -> `const NameHardenBouncer = "Harden-Bouncer"`
- `plan:internal/loomshed#NameHardenBurlerReview` -> `const NameHardenBurlerReview = "Harden-Burler-Review"`
- `plan:internal/loomshed#NameHardenBurlerFix` -> `const NameHardenBurlerFix = "Harden-Burler-Fix"`
- `plan:internal/reportengine#hardenerPhases` -> `func hardenerPhases() map[string]string`
- `plan:internal/loomrecipe#TestHardenerRecipe` -> `func TestHardenerRecipe(t *testing.T)`

**Edit:**
- `internal/shedrun#RecipeNames`
- `internal/shedrun#TestRecipeNames`
- `internal/shedcli#recipes`
- `internal/shedcli#TestRecipes_KeySet`
- `internal/battencli#childRecipeSupported`
- `internal/battencli#unsupportedChildRecipeRefusal`
- `internal/battencli#TestChildRecipeSupported`
- `internal/loomrecipe#embeddedRecipes`
- `internal/loomrecipe#RecipeEngines`
- `internal/loomrecipe#BouncerRunSubdir`
- `internal/loomrecipe#IsFinalizeRow`
- `internal/loomrecipe#TestRecipeEngines_ReportsExactlyLoomsOwnEngineSet`
- `internal/loomrecipe#TestBouncerRunSubdir`
- `internal/loomrecipe#TestIsFinalizeRow`
- `internal/loomrecipe#TestRowNames_FollowEachRecipesRowOrder`
- `internal/loomrecipe#TestInterruptPolicies_MatchAssembledRows`
- `internal/loomrecipe#wantProducerTable`
- `internal/loomshed#InterruptPolicies`
- `internal/shedrecipe_test#coverageGuardAllowedUnreachableEngines`
- `internal/reportengine#runRecipes`
- `internal/reportengine#recipePhaseTables`
- `internal/reportengine#TestCompute_SelectsRuns`
- `internal/reportengine#TestComputeTiming`

**Extends:**
- `internal/shedrun/doc.go`
- `internal/loomrecipe/doc.go`
- `internal/loomshed/doc.go`
- `internal/reportengine/doc.go`

**Uses:**
- `contracts/recipes#DarnRecipe`
- `internal/shedcli/doc.go`

**Intent:**
The `hardener` loom recipe and its place in every closed recipe list (D1, D3).

`hardener-recipe.yaml` declares `contract: loom` and `verify_source: tree_scope`, entry Preflight, sole terminal Friction-Reflect, and no Drift rows; its header comment states the shape as darn's does.
Rows in on_done order:
- Preflight → Record-Write → Harden-Bouncer.
- Harden-Bouncer, engine `Bouncer`, segment `Harden-Review`, on_stuck Harden-Burler-Review, on_done Describe; config `run_subdir: harden`, `role_prefix: harden`, `subject: code`, `reviewer_fan_setting: hardener.reviewer_fan`, `auto_accept_setting: hardener.auto_accept`, `artifact_paths: [_lym/discussion/decision-record.md]`, `rubric_stencil: harden-rubric-review`, `commit_seam: harden`, `carry_over: Harden-Review`, `harden_round: true`.
- Harden-Burler-Review, engine `BurlerReview`, segment `Harden-Review`, on_stuck and on_done Harden-Burler-Fix; config `run_subdir: harden`, `role_prefix: harden`, `reviewer_fan_setting: hardener.reviewer_fan`, `fixer_fan_setting: hardener.fixer_fan`, `harden_round: true`; its profile has `tool-use: true`, `fix-scope: source`, `language-skills: true`, `rubric_stencil: harden-rubric-review`, a target whose instructions name the charter's code paths in the decision record, and the decision record as fasit.
- Harden-Burler-Fix, engine `HardenFix`, segment `Harden-Review`, on_stuck and on_done Harden-Bouncer; config `run_subdir: harden`, `role_prefix: harden`, `fixer_fan_setting: hardener.fixer_fan`, `review_row: Harden-Burler-Review`, profile `fix-scope: source`, `language-skills: true`; no `gates` key.
- Describe → PR-Review → Finalize → Friction-Reflect.
The fix row is the recipe's one build row, placed for the contract's order clause at Harden-Bouncer (card 4).
`recipes.go` embeds it as `HardenerRecipe`.

Every closed list admits `hardener`:
- `shedrun`: the `RecipeHardener` constant and `RecipeNames`.
- `shedcli.recipes`: an entry armed by `loomcli.ArmAt` with the five generic verbs and `loomcli.BootstrapVerb`.
- `battencli.childRecipeSupported` and the names its refusal lists, so batten's Run-Shed starts a board task whose `recipe` is `hardener` through dobby.
- `loomrecipe`: `embeddedRecipes`, the `RecipeEngines` recipe list, and the `BouncerRunSubdir` and `IsFinalizeRow` recipe lists, so a capped Harden-Bouncer's `lym loom circling` verb finds its run directory.
- `reportengine`: `runRecipes`, and `recipePhaseTables` with `hardenerPhases`, which maps Preflight and Record-Write to setup, the three Harden rows to review, Describe to write, and PR-Review, Finalize and Friction-Reflect to publish-finalize.
- `loomshed.InterruptPolicies` gains the three ring rows with the policies the Code-Review perch's rows carry, and the meta test `TestInterruptPolicies_MatchAssembledRows` walks the hardener recipe too; `wantProducerTable` gains the hardener rows' engines.
`loomshed` names the three ring rows, as it names every pinned row.
`RecipeEngines` now reaches `HardenFix`, so `coverageGuardAllowedUnreachableEngines` drops the entry card 11 added.

Tests, all tier0:
`TestRecipeNames`, `TestRecipes_KeySet` and `TestChildRecipeSupported` gain `hardener`;
`TestRecipeEngines_ReportsExactlyLoomsOwnEngineSet` gains `HardenFix`;
`TestBouncerRunSubdir` gains Harden-Bouncer → `harden`; `TestIsFinalizeRow` gains the hardener Finalize row;
`TestRowNames_FollowEachRecipesRowOrder` gains the hardener row order;
`TestInterruptPolicies_MatchAssembledRows` gains the hardener recipe;
`TestCompute_SelectsRuns` selects a hardener run and `TestComputeTiming` gains a hardener row's phases.
`TestHardenerRecipe` parses and expands the recipe and checks:
- it passes the loom contract;
- its on_done path from the entry is Preflight, Record-Write, Harden-Bouncer, Describe, PR-Review, Finalize and Friction-Reflect, and the ring's on_stuck edges run Harden-Bouncer → Harden-Burler-Review → Harden-Burler-Fix → Harden-Bouncer;
- the review row's profile sets `tool-use: true`, `fix-scope: source` and `rubric_stencil: harden-rubric-review`, and the fix row carries no `gates` key;
- its routing, projected with a Harden-Review budget, gives the three ring rows the segment and its budget.

**ImpactSummary:** One new embedded recipe and one name added to each closed recipe list, interrupt policy and pinned row table; every existing recipe's rows and readers are unchanged.

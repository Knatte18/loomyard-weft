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
- `internal/battenshed/doc.go`
- `docs/overview.md`
- `internal/boardengine/task.go`
- `internal/driftcli/wiring.go`
- `internal/driftcli/cli.go`
- `internal/driftcli/doc.go`
- `internal/landingshed/fixerrun.go`
- `internal/landingshed/handback.go`
- `internal/landingshed/parentalign.go`
- `internal/landingshed/finalize.go`
- `internal/landingshed#Deps`
- `internal/landingshed/doc.go`
- `internal/arbiterstop/doc.go`
- `internal/arbiterstop/arbiterstop.go`
- `internal/sheddriver/doc.go`
- `internal/battenshed/innerrun_binaryrestep.go`
- `internal/battenshed#InnerRunDeps`
- `internal/battencli#battenCLI.wire`
- `internal/battencli#restepSkips`
- `internal/loomcli/settledguard.go`
- `internal/loomcli/subjects.go`
- `internal/loomcli/drivewiring.go`
- `contracts/specs/loom-status-spec.md`
- `contracts/specs/landing-fixer-report-spec.md`
- `contracts/specs/loom-final-summary-spec.md`
- `contracts/stencils/loom/loom-rubric-code-review.md`
- `contracts/stencils/stencils.go`
- `contracts/orchrole/orch-role.md`
- `internal/boardcli#storeVerbs`
- `internal/loomcli#arbiterRecipeDescription`
- `internal/loomcli#TestRecipeKeyedWiring`
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
- `internal/boardcli/doc.go`
- `internal/boardengine/doc.go`
- `internal/battencli#TestWire_EverySeamFilled`
- `internal/loomcli#TestLandingDeps_EveryFieldPopulated`
- `internal/loomcli#TestWire_BeamRecipe`
- `internal/loomcli#TestWire_DarnRecipe`
- `internal/loomcli#TestWire_EverySeamFilled`
- `internal/loomcli#TestWire_SamplerRecipe`
- `internal/testkit/envkit#TestFullEnv_FillsEverySeam`
- `internal/testkit/envkit#TestNilSeams_ReportsExactPaths`
- `internal/testkit/envkit#TestNilSeams_SkipsNilLegalPaths`

**Intent:** Make arras a seedable recipe accepted and wired wherever tapestry is accepted by name (D10); motif stays out of every one of these sites (D9).

- `shedrun.RecipeNames` gains `RecipeArras` in sorted position (arras first); `ValidateRecipe` follows, so a board task's `recipe` field, `lym shed seed --recipe` and batten's `WriteSeed` accept `arras`; `doc.go` names the vocabulary with arras and says motif, the recipe that makes a sketch, the decision record, is a block source only.
- shedcli's `recipes` table gains an `arras` entry armed as tapestry's is (`loomcli.ArmAt`, the same verbs, `loomcli.BootstrapVerb`); `doc.go`'s recipe list gains it.
- battencli's `childRecipeSupported` admits `RecipeArras`, and `unsupportedChildRecipeRefusal` names it among the admitted recipes and the task types; its doc's "four" becomes "five".
  `childSeedParams` follows, giving arras the parent param tapestry gets, and its doc names arras among the recipes that declare one; `battenshed.ErrUnsupportedChildRecipe`'s doc names arras among the recipes `lym loom start` serves.
- battenshed's `doc.go` names arras beside tapestry, darn and sampler in its escalate-halt sentence.
- The board's `recipe` field help text in `storeVerbs` lists arras beside darn and tapestry.
- loomcli's `arbiterRecipeDescription` names a recipe with a Plan-Write row by `recipeLabel(recipe)` instead of the literal tapestry, with "an" before a label opening with a vowel, so an arras run reads "This is an arras run, a task driven from discussion to landing" and a tapestry run reads as before.
- reportengine's `runRecipes` gains arras, and `recipePhaseTables` gives arras tapestry's phase table with the `Motif` row added to the discussion phase; `doc.go` and `runs.go`'s file comment name arras beside tapestry and darn among the selected recipes.
- gateslot's `doc.go` and the PR-Review stencil's leading comment list arras among the recipes that run PR-Review.
- tokencount's `countedRecipes` gains arras, so the default selection counts finished arras runs; `recipeList` renders a list of three or more as `a, b <conjunction> c`, and `main.go`'s package comment names arras beside tapestry and darn.
- `docs/overview.md` names arras in the loomrecipe line, the shedcli arming-table line and the list of loom recipes the `lym loom` verbs serve, and names motif as a block source in the loomrecipe line.
  It also names arras among the loom runs whose fixer's escalate halts with `EscalateStopPrefix`, among the child recipes in the batten bullet, and among the task runs batten's `Run-Shed` row seeds.
- boardengine's refusal of a task carrying the project recipe names arras among a task's recipes, in its statement and its way forward; driftcli's no-run-status refusal, its `cli.go` help text and its `doc.go` name arras beside tapestry and darn as the runs whose worktree the drift verbs run in.
- Every other comment, doc, spec and stencil that lists the loom recipes by name names arras beside tapestry: the escalate-halt and `EscalateJudgedFirst` sentences in landingshed, arbiterstop, sheddriver, battenshed's binary re-step and `InnerRunDeps.HandBack`, battencli's `HandBack` wiring and its blocked-stop doc, and loomcli's drive wiring; the run kinds in loomcli's settled guard and subjects file comments; `loom-status-spec.md`, `landing-fixer-report-spec.md` and `loom-final-summary-spec.md`; the Code-Review rubric's leading comment and its `stencils.go` doc; `orch-role.md`'s batten-run and escalate-halt sentences; and gateslot's `doc.go` slot table rows for Code-Burler-Fix, Drift-Verify, PR-Review and Finalize.
The board's `default_recipe` is untouched.

Tests, all tier0:
- `TestRecipeNames` and `TestRecipes_KeySet` (its literal want and the `//testtiming:keep` reason's list) gain arras.
- `TestValidateRecipe` gains rows: `arras` accepted, `motif` refused naming the available recipes.
- `TestWire_WriteSeedRefusesANonLoomChildBeforeTouchingTheWorktree`'s admitted loops gain arras, and the refusal's text check follows the new wording; `TestChildSeedFor_CarriesRecipeDriverAndParams` gains an arras row carrying `params.parent`.
- `TestRecipeKeyedWiring` gains an Arras row: plan source, a plan directory, and the description fragment "a task driven from discussion to landing" with the label arras.
- `TestCompute_SelectsRuns` gains an arras run selected as a tapestry run is; `TestComputeTiming` gains an arras run whose Motif row counts under the discussion phase.
- `TestRecentRuns` gains a finished arras run counted by the default selection, and asserts the three-recipe sentence.

**ImpactSummary:** Additive entries in five name-keyed tables, three refusal texts, one description label and the docs, comments, specs and stencils that list recipe names; no existing recipe's acceptance or wiring changes.

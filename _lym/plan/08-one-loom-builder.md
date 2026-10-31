# Card 8 — one-loom-builder

**Edit:**
- `internal/loomcli/doc.go`
- `internal/loomrecipe#`
- `internal/loomcli#loomCLI.armAt`
- `internal/loomcli#loomCLI.recipeRouting`
- `internal/loomcli#loomCLI.newShed`
- `internal/loomcli#loomCLI.specFor`
- `internal/loomcli#loomCLI.wireDarn`
- `internal/loomcli#loomCLI.wire`
- `internal/loomcli#recipeRow`
- `internal/loomcli#TestRenameRetiredRows`
- `internal/loomcli#TestWire_DefaultConfig`
- `internal/reportengine#darnPhases`

**Uses:**
- `internal/reportengine/doc.go`

**Intent:** `New` and `NewDarn`, `Routing` and `DarnRouting`, and `TapestryRetiredRows` and `DarnRetiredRows` in `internal/loomrecipe` become one builder each, taking the recipe name, with each recipe's retired rows kept as that recipe's data (D8). The card changes the whole package, so it names the package.

- `New(recipe string, env shedrecipe.Env, paths shedbuild.ShedPaths) (*shedengine.Shed, error)` builds the embedded loom recipe the name selects (`parseEmbeddedRecipe`, empty reading as tapestry): the status-path coherence checks, the review segments' budgets and `zero_rounds` stamp, and, for a recipe whose `verify_source` is `tree_scope`, the `DarnVerifyAttempts` check and stamp onto the verify gate of its build row (the row whose engine is in `shedrecipe.LoomBuildEngines`) and of its Code-Review segment's BurlerFix row, the rows `darnVerifyGatedRows` names today, then `shedbuild.NewShedFrom` and the review budget on the built rows. A recipe that does not declare `contract: loom` is refused naming `NewPawl` or `lym shed` as the way forward, a fail-loud wiring guard in `refusal-spec.md`'s sense that takes no reaching test: its one caller, `newShed`, branches pawl off first, and card 19 refuses every other non-loom recipe in `armAt` before any wiring; pawl keeps `NewPawl`/`PawlRouting` (D2's exception).
- `Routing(recipe string, budgets map[string]int) (shedengine.Routing, error)` is the routing projection of the same recipe with the budgets applied, as `Routing` and `DarnRouting` do today.
- `NewDarn`, `DarnRouting`, `TapestryRetiredRows`, `DarnRetiredRows` and `darnVerifyGatedRows` go; `RetiredRows(recipe)` (card 7) is the one retired-row source.
- `IsFinalizeRow` and `BouncerRunSubdir` keep their two-recipe scan unchanged.
- Every package test that called the old builders calls the new ones with the recipe name, the darn tests passing `shedrun.RecipeDarn`.

Callers outside the package: `armAt` renames retired rows through `RetiredRows(c.recipe)` with no branch, a nil map renaming nothing; `recipeRouting` and `newShed` keep the pawl branch and otherwise call `Routing(c.recipe, …)`/`New(c.recipe, …)`; `wireDarn`'s segment-bounces closure and `wire`'s call `Routing(c.recipe, …)`; `recipeRow` maps a retired producer through `RetiredRows` of tapestry and of darn; `darnPhases` reads `RetiredRows(shedrun.RecipeDarn)`; `specFor`'s comment names `New`. `TestRenameRetiredRows` and `TestWire_DefaultConfig` call the new forms. The package doc describes the one builder and the per-recipe retired-row data.

**ImpactSummary:** Six exported builders collapse into three recipe-keyed ones; every caller is in loomcli, reportengine or the package's own tests, and the built sheds are unchanged.

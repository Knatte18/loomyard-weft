# Card 18 — recipe-keyed-wiring

**Create:**
- `plan:internal/loomcli#TestRecipeKeyedWiring` -> `func TestRecipeKeyedWiring(t *testing.T)`

**Edit:**
- `internal/loomcli#loomCLI.verifySource`
- `internal/loomcli#loomCLI.specFor`
- `internal/loomcli#loomCLI.wire`
- `internal/loomcli#loomCLI.wireLightweight`
- `internal/loomcli#landingDeps`
- `internal/loomcli#fixerPlanDir`
- `internal/loomcli#fixerTestFileMatcher`
- `internal/loomcli#arbiterRecipeDescription`
- `internal/loomcli#tacklerFor`
- `internal/loomcli#TestSpecFor_RestepHold`
- `internal/loomcli#TestTacklerFor`
- `internal/loomcli#TestFixerTestFileMatcher`
- `internal/loomcli#TestWire_DarnRecipe`
- `internal/loomcli/doc.go`

**Delete:**
- `internal/loomcli#codeReviewOff`

**Uses:**
- `plan:internal/loomrecipe#IsLoom`
- `plan:internal/loomrecipe#HasEngine`
- `plan:internal/loomrecipe#VerifySource`

**Intent:** Every remaining branch on `RecipeDarn` or `RecipeTapestry` in `internal/loomcli` reads the recipe's rows or keys instead, and the verify source follows the recipe's `verify_source` key (D8). Pawl's branches stay, under D2's exception. Each read goes through card 7's accessors on `c.recipe`, an empty recipe read as tapestry; an accessor error is returned where the caller returns errors, and elsewhere reads as the tapestry answer.

- `verifySource`: `verify_source: tree_scope` takes `darnVerifySource`, and `plan` or none (pawl) takes `planVerifySource`.
- `specFor`'s `RestepHold`, and `landingDeps`' `EscalateJudgedFirst`, hold for a recipe that declares the loom contract (`IsLoom`).
- `wire` calls `wireDarn` for a recipe with a `DarnWrite` row. It tells the review rows, and `wireLightweight` the plan context, the record path as `PatternRecordPath` for every loom recipe, so a darn run's Code-Review perch checks the record's declared PATTERN changes landed (D3); pawl stays told none.
- `landingDeps`' failure tracer, `fixerPlanDir` and `fixerTestFileMatcher` key on a `PlanWrite` row: with one, the plan's trace, directory and language; without, none and the any-language matcher. `codeReviewOff` goes: `landingDeps` sets `CodeReviewOff` to `codeBudgetZero(l)` for every recipe, since only the Parent-Align row reads the seam and a recipe without that row never builds one, so its darn branch was dead.
- `arbiterRecipeDescription` and `tacklerFor` keep pawl's description. For a loom recipe, they describe a recipe with a `PlanWrite` row as the tapestry run, "a task driven from discussion to landing", and any other as "a task's code change driven from its decision record through one build session, its review and landing".

`TestRecipeKeyedWiring` (tier0) drives the pure branches per recipe name: `verifySource` picks the tree-scope source for darn and the plan source for tapestry and pawl; `fixerPlanDir` is set for tapestry and empty for darn and pawl; `arbiterRecipeDescription` differs for tapestry, darn and pawl. `TestRecipeKeyedWiring` also calls `landingDeps` per recipe: `EscalateJudgedFirst` holds for tapestry and darn and not for pawl, and `TraceFailure` is set for tapestry alone. `TestWire_DarnRecipe` asserts a darn seed's review rows are told the record as `PatternRecordPath`. `TestSpecFor_RestepHold`, `TestTacklerFor` and `TestFixerTestFileMatcher` take the same outcomes with the recipe-keyed reads.

**ImpactSummary:** Ten loomcli branches switch, and codeReviewOff and its dead darn branch go, from recipe names to recipe keys and rows with unchanged outcomes for tapestry, darn and pawl, except that darn's review rows are now told the record's PATTERN declarations.

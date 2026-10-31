# Card 6 — sampler-recipe

**Create:**
- `plan:internal/shedrun#RecipeSampler` -> `const RecipeSampler = "sampler"`
- `plan:internal/loomrecipe#EngineRow` -> `func EngineRow(recipe, engine string) (string, bool, error)`
- `plan:internal/loomrecipe#TestSamplerRecipe` -> `func TestSamplerRecipe(t *testing.T)`

**Edit:**
- `internal/shedrun#RecipeNames`
- `internal/shedrun#TestValidateRecipe`
- `internal/shedrun#TestRecipeNames`
- `internal/shedcli#recipes`
- `internal/shedcli#TestRecipes_KeySet`
- `internal/shedcli#TestRecipes_Entries`
- `internal/loomrecipe#embeddedRecipeSource`
- `internal/loomrecipe#verifyGatedRows`
- `internal/loomrecipe#RecipeRow`
- `internal/loomrecipe#Rows`
- `internal/loomrecipe#RowNames`
- `internal/loomrecipe#AmendmentTarget`
- `internal/loomrecipe#TestDarnVerifyBudget`
- `internal/loomrecipe#TestRowNames_FollowEachRecipesRowOrder`
- `internal/loomrecipe#TestRecipeKeys`
- `internal/loomrecipe#TestInterruptPolicies_MatchAssembledRows`
- `internal/loomshed#InterruptPolicies`
- `internal/loomrecipe/doc.go`
- `internal/shedcli/doc.go`
- `docs/overview.md`

**Uses:**
- `plan:contracts/recipes#SamplerRecipe`
- `internal/shedrun/doc.go`
- `internal/loomrecipe#testEnv`
- `internal/loomrecipe#TestDriftRows_EntryGate`

**Intent:**
D1 and D2: the `sampler` recipe's name in the closed recipe vocabulary, and its reach into `lym shed` and loomrecipe's accessors.
The recipe file `contracts/recipes/sampler-recipe.yaml`, embedded as `recipes.SamplerRecipe`, is card 5's, which describes its rows.
`shedrun.RecipeSampler` joins the closed vocabulary and `RecipeNames` returns `batten, darn, pawl, sampler, tapestry`; `TestValidateRecipe` gains an accepted `sampler` row and `TestRecipeNames` the new list.
`shedcli`'s `recipes` gains `"sampler"` armed by `loomcli.ArmAt` with darn's verbs and `loomcli.BootstrapVerb`; `TestRecipes_KeySet`'s literal row becomes `batten, darn, pawl, sampler and tapestry`, and `TestRecipes_Entries` gains its row; `shedcli/doc.go` names the recipe in its table sentence.
`loomrecipe.embeddedRecipeSource` maps `shedrun.RecipeSampler` to `recipes.SamplerRecipe`, the `Rows` and `RowNames` docs both name `shedrun.RecipeSampler` as the sampler recipe, and the `AmendmentTarget` doc names `shedrun.RecipeSampler` in its list of recipe names, since the function now answers for it instead of refusing it; `loomrecipe/doc.go`'s sentences on the recipes `Rows`, `RowNames` and `AmendmentTarget` serve name sampler too.
`RecipeRow` gains `OnDone string`, filled by `Rows`, and the new `EngineRow` returns the name of the first row of the named recipe running `engine`, false when none, refusing an unknown recipe as `HasEngine` does.
`verifyGatedRows` keeps only the candidate rows that carry a `verify` gate entry, so on the sampler recipe it returns `Code-Burler-Fix` alone and the Sampler-Session row, which has no gates, gets no stamp; on darn it still returns Darn-Write and Code-Burler-Fix; `applyVerifyAttempts`' refusal of a named row with no verify entry stays as a fail-loud guard.
`InterruptPolicies` maps `NameSamplerSession` to `InterruptPolicyReinvoke`, with a comment that the row reattaches the live session or resumes the recorded one rather than spawning a second.
Tests: `TestSamplerRecipe` builds the recipe with `New` over `testEnv` (the contract check passes) and asserts the on_done path from the entry is Preflight, Record-Write, Sampler-Session, Code-Bouncer, Describe, PR-Review, Finalize, Friction-Reflect; that the three Code-Review rows carry an entry gate which, with `env.Setting` answering `sampler.code_review` false as the template does, does not hold, and with true holds (a gate that does not hold is recorded Skipped and routes on to the row's on_done, Code-Bouncer's being Describe, with no producer called, which `internal/shedengine#TestStep_EntryGate` pins for any row), while `Describe`'s gate is nil also under `Env.Segments["Code-Review"].MaxBounces` 0 (D1, D2, D3 acceptance: the perch off skips every Code-Review row and Describe still runs; on, the perch runs with loom's `code_*` budget, which the rows' `MaxBounces` carry); that `Finalize`'s reroute names Code-Bouncer; and that `EngineRow(sampler, "SamplerSession")` answers `Sampler-Session` and `EngineRow(darn, "SamplerSession")` answers none.
`TestDarnVerifyBudget` gains a subtest: on the parsed sampler recipe `verifyGatedRows` is `[Code-Burler-Fix]`, and `applyVerifyAttempts` with 7 sets Code-Burler-Fix's verify attempts to 7 and leaves Describe's description gate at 3, read off the parsed rows as the darn subtest reads them (D6 acceptance on the budget).
A sampler run's empty landing (D1, D6: an agent that wrote only a board note, with no code commit, passes Describe's gate and lands with no pull request) needs no new code: the description gate checks the file's format only, pinned by `internal/summaryparser#TestValidateDescription_Table`'s `ok` row, and a task branch with no commits beyond its parent resolves to no pull request, pinned by `internal/landingshed#TestResolvePullRequest`'s no-commits rows; Record-Write's commits land under `_lym`, never on the code branch, so a note-only run has none.
`TestRowNames_FollowEachRecipesRowOrder` gains a sampler row and `OnDone: row.OnDone` in its `wantRow` literal, since `Rows` now fills it for every recipe, `TestRecipeKeys` a sampler row (loom, verify_source `tree_scope`, record engine RecordWrite), and `TestInterruptPolicies_MatchAssembledRows` builds the sampler recipe beside tapestry and darn.
`loomrecipe/doc.go` describes the sampler recipe's row path and that New and Routing serve it; `docs/overview.md` gains a paragraph on the sampler recipe beside darn's and adds it to the lines listing the loom recipes and the `lym shed` table.

**ImpactSummary:** the embedded sampler recipe reaches the vocabulary, `lym shed`'s table and loomrecipe's accessors; `verifyGatedRows` narrows to rows carrying a verify gate, which leaves darn's two rows unchanged, `RecipeRow` gains one field its readers ignore, and `InterruptPolicies` gains a Sampler-Session entry.

**Commit:** 6: the sampler loom recipe

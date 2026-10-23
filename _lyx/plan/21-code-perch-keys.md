# Card 21 — code-perch-keys

**Create:**
- `plan:internal/loomengine#retiredPerchKeyRenames` -> `var retiredPerchKeyRenames []yamlengine.KeyRename`
- `plan:internal/loomengine#PerchCode` -> `const PerchCode Perch = "code"`

**Delete:**
- `internal/loomengine#PerchWebster`

**Edit:**
- `internal/loomengine#Config`
- `internal/loomengine#Config.Caps`
- `internal/loomengine#reviewPerches`
- `internal/loomengine#ConfigOpenMaps`
- `internal/loomengine#ConfigSpec`
- `internal/loomengine#LoadConfig`
- `internal/loomengine#fanKeys`
- `internal/loomengine#segmentModelKeys`
- `internal/loomengine#MigrateConfig`
- `internal/loomengine#ReviewSettings`
- `internal/loomengine#JudgeSettings`
- `internal/loomengine#ResolveReview`
- `internal/loomengine#ResolveJudge`
- `internal/loomengine/template.yaml`
- `internal/loomengine#templateConfig`
- `internal/loomengine#TestLoadConfig_Loads`
- `internal/loomengine#TestLoadConfig_Refuses`
- `internal/loomengine#TestMigrateConfig_RetiredKeys`
- `internal/loomengine#TestResolveReview`
- `internal/loomengine#TestResolveJudge`
- `internal/loomengine#TestResolveReviewAndJudge_MalformedSpec`
- `internal/configsync#formerLoomValues`
- `internal/runprofile/template.yaml`
- `contracts/recipes/loom-recipe.yaml`
- `internal/loomcli#loomCLI.wire`
- `internal/loomcli#loomPerches`
- `internal/loomcli#loomSegmentPerches`
- `internal/loomcli#loomSegmentSettings`
- `internal/loomcli#TestWire_ReviewKeysReachTheEnv`
- `internal/configreg#TestLookup`
- `internal/configreg#TestModules_EveryTemplateKeyClassified`
- `internal/configreg#TestPrepareRunSettings`
- `internal/configcli#TestConfigOverRealHub`
- `internal/hubreconcile_test#loomFormerUnits`
- `internal/hubreconcile_test#assertBoardStrippedOnce`
- `internal/loomrecipe#TestEscalation_PerchBudgetOneWithAutoAccept`
- `internal/loomrecipe#TestShippedRecipes_FanSettingsAgreeWithinSegment`

**Uses:**
- `internal/loomengine#retiredReviewKeyRenames`

**Intent:** Rename the Webster-Review perch's `loom.yaml` keys to the Code-Perch's, `webster_*` to `code_*` with the same defaults (D5, D14): `webster_reviewer_model`, `webster_fixer_model`, `webster_reviewer_fan`, `webster_fixer_fan`, `webster_judge`, `webster_seed`, `webster_max_bounces`, `webster_hard_max_bounces` and `webster_auto_accept` become `code_reviewer_model`, `code_fixer_model`, `code_reviewer_fan`, `code_fixer_fan`, `code_judge`, `code_seed`, `code_max_bounces`, `code_hard_max_bounces` and `code_auto_accept`.
`PerchWebster` becomes `PerchCode`, whose value `code` is the key prefix `reviewbudget` derives the budget keys from, so `Caps`, `reviewPerches`, `ConfigSpec`'s categories, `LoadConfig`'s budget check, `fanKeys` and `segmentModelKeys` follow; `Config`'s nine `Webster*` fields become `Code*`, and `ReviewSettings` and `JudgeSettings` rename their `Webster` members to `Code` (`WebsterKeys` to `CodeKeys`).
The template's comments name the Code-Review perch; `review_max_bounces`' comment names it in the perch list.

`MigrateConfig` gains one rename table, `retiredPerchKeyRenames`, applied through `yamlengine.RenameKeys` after `copyWebsterModelsOntoMerge` and `retiredReviewKeyRenames`, so a retired `webster_review` first becomes `webster_reviewer_model` and then `code_reviewer_model`, and a document holding both names keeps the new one; the D14 retirements of later cards append to the same table.
This is the migration `configreg` registers as loom's `Migrate`, so `lyx config reconcile`, `hubreconcile.Ensure`, the profile check and run-settings layering all read a `webster_*` key as its `code_*` successor (D14).
`configsync`'s former loom values and the run-profile template name the `code_*` keys; the hub's `profiles.yaml` is never rewritten.
The loom recipe's Webster-Review rows read `loom.code_reviewer_fan`, `loom.code_fixer_fan` and `loom.code_auto_accept`; their row and segment names change with the recipe card.
loomcli's perch tables map the Webster-Review segment and Bouncer row to `PerchCode`, and `wire` and `loomSegmentSettings` read `Code` from the resolved settings, the darn Bouncer pair included.

The tests switch to the new keys and fields; `TestMigrateConfig_RetiredKeys` gains rows moving each `webster_*` key to its `code_*` successor, one where both are present, and one chaining `webster_review` through to `code_reviewer_model`.

**ImpactSummary:** Nine loom.yaml keys and one perch constant renamed, with their readers in loomengine, loomcli, configsync, the loom recipe and the run-profile template, and a migration so no hub file or profile needs an operator edit.

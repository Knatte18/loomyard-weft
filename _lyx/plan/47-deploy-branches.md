# Card 47 — deploy-branches

**Create:**
- `plan:internal/landingshed#retiredKeyRenames` -> `var retiredKeyRenames []yamlengine.KeyRename`

**Edit:**
- `internal/landingshed#Config`
- `internal/landingshed#ConfigSpec`
- `internal/landingshed#MigrateConfig`
- `internal/landingshed/template.yaml`
- `internal/landingshed#newTestDeps`
- `internal/landingshed#TestLoadConfig_TemplateDefaults`
- `internal/landingshed#TestLoadConfig_InvalidKeys`
- `internal/landingshed#TestLoadConfig_OptionalStringKeys`
- `internal/landingshed#TestMigrateConfig`
- `internal/battenshed#LandingSettings`
- `internal/battenshed#landProducer.pullAndDeploy`
- `internal/battenshed#newLandFixture`
- `internal/battenshed#TestLand_Dispositions`
- `internal/battencli#landDeps`
- `internal/configcli#buildConfigLong`
- `internal/configengine_test#TestLoad`
- `internal/configreg#TestModules_LocalLayerRefusals`
- `internal/configsync#TestReconcileHubWideAt_SeedsFromHubPrimeLegacyOrTemplate`
- `internal/loomcli#TestWire_DefaultConfig`
- `internal/loomrecipe#buildSequenceFixture`
- `internal/yamlengine#MissingKeys`
- `internal/yamlengine#sequenceBasePath`
- `internal/yamlengine#applyExistingSequences`
- `internal/yamlengine#listKeyTemplate`
- `internal/yamlengine#TestMissingKeys`
- `internal/yamlengine#TestReconcile`
- `internal/yamlengine#TestSetValues`
- `internal/yamlengine#TestSetValues_Refusals`

**Extends:**
- `internal/battenshed/doc.go`

**Intent:** `landing.yaml`'s `require_pr_to_base` becomes `deploy_branches`, default `["main"]`, read only by batten's Land row to decide whether to deploy; an empty list deploys nothing (D13).
`Config.RequirePRToBase` becomes `DeployBranches`, the template key and its comment name the deploy, and `ConfigSpec` classifies the new key as `CategoryCrossPair`, as the old one was.
`landingshed.MigrateConfig` moves `require_pr_to_base` to `deploy_branches` keeping its value, an empty list included, through `yamlengine.RenameKeys` over the new `retiredKeyRenames` table, after the `co_authored_by` move; a document already holding `deploy_branches` keeps it and drops the old key (D14).
`yamlengine`'s reconcile keeps treating `deploy_branches: []` as a meaningful empty list, as it treated `require_pr_to_base: []` (the plan writer's note); its comments and tests name the new key.
Batten's `LandingSettings.RequirePRToBase` becomes `DeployBranches`, which `pullAndDeploy` reads, and battencli's land seam copies it.
The deploy now runs every landing's code with the Code-Perch, or Darn-Review, as its last review where its perch budget is above 0, and Finalize's capped fix unreviewed; a run whose perch budget is 0 deploys code with no review at all, bounded only by the landing gate, the operator's `deploy_branches` list and D10's size and kind caps over every fixer change it lands.
`lyx config`'s help example names the new key.

`TestMigrateConfig` pins the move with a list value, with an empty list kept empty, and with both keys present; `TestLand_Dispositions` pins a deploy for a parent in the list and none for one outside it or for an empty list.

`doc.go`'s Land section names the key and the deploy's review bound.

**ImpactSummary:** One landing.yaml key renamed with a migration; its only runtime reader is batten's Land row, and the rest are tests and help text.

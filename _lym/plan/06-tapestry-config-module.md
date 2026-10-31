# Card 6 — tapestry-config-module

**Create:**
- `internal/loomengine/tapestry.yaml`
- `internal/loomengine#TapestryConfig`
- `internal/loomengine#TapestryConfigTemplate`
- `internal/loomengine#TapestryConfigOpenMaps`
- `internal/loomengine#TapestryConfigSpec`
- `internal/loomengine#LoadTapestryConfig`
- `internal/loomengine#ValidateTapestryLocalValues`
- `internal/loomengine#Config.WithTapestry`
- `internal/loomengine#TestLoadTapestryConfig`

**Edit:**
- `internal/loomengine#Config`
- `internal/loomengine#ConfigSpec`
- `internal/loomengine#ConfigOpenMaps`
- `internal/loomengine#LoadConfig`
- `internal/loomengine#ValidateLocalValues`
- `internal/loomengine#fanKeys`
- `internal/loomengine#segmentModelKeys`
- `internal/loomengine#reviewPerches`
- `internal/loomengine#loomSegmentModelSource`
- `internal/loomengine#ResolveReview`
- `internal/loomengine#TestLoadConfig_Loads`
- `internal/loomengine#TestLoadConfig_Refuses`
- `internal/loomengine#TestResolveReview`
- `internal/loomengine/template.yaml`

- `internal/configreg#Modules`
- `internal/configreg#KeyMoves`
- `internal/configreg#TestModules_EveryTemplateKeyClassified`
- `internal/configreg#TestPrepareRunSettings`
- `internal/configsync#formerLoomValues`
- `internal/configsync#formerTemplateValues`
- `internal/configsync#TestReconcile_MovesKeysAcrossModules`
- `internal/loomcli#loomSegmentSettings`
- `internal/loomcli#perchHardCap`
- `internal/loomcli#loomCLI.reviewBudget`
- `internal/loomcli#loomCLI.wire`
- `internal/loomcli#TestWire_DefaultConfig`
- `internal/loomcli#TestWire_DarnRecipe`
- `contracts/specs/implementer-spec.md`
- `internal/loomrecipe#TestShippedRecipes_FanSettingsAgreeWithinSegment`
- `contracts/recipes/tapestry-recipe.yaml`
- `docs/overview.md`
- `internal/loomcli#TestPerchHardCap`
- `internal/configcli#TestDispatchLayers`
- `internal/shedbuild#engineMinimalConfig`
- `internal/shedrecipe#implementerEntry`
- `internal/shedrecipe#TestImplementerEntry_RefusesAnUnusableSetting`
- `internal/shedrecipe#planWriteEntry`
- `internal/shedrecipe/doc.go`
- `internal/configreg#TestSettingLookup`
- `internal/configreg#TestModules_LocalLayerRefusals`
- `internal/testkit/envkit#TestFullEnv_FillsEverySeam`
- `internal/battencli#TestPrepareRunSettings`
- `internal/shedbuild#TestBuild_EntryGate`
- `internal/shedbuild#TestBuild_Errors`
- `internal/loomrecipe#TestReviewBudget_NegativeRefused`
- `internal/configcli/doc.go`
- `internal/boardcli#Command`
- `internal/reviewbudget#Values`
- `internal/reviewbudget#CheckResolved`
- `internal/reviewbudget#TestCheckResolved`
- `internal/loomengine#TestConfigTemplate_ContainsEveryConfigYAMLTag`
- `internal/loomengine#TestConfigTemplate_ImplementerDefaultsToWebsterAndAcceptsALocalValue`
- `internal/loomengine#TestMigrateConfig`
- `internal/loomengine#twillLenses` -> `func twillLenses(dirs configengine.Dirs, fan, module, key string) ([]twilldirective.Lens, error)`
- `internal/loomengine#planLensDirective`
- `internal/loomengine#discussionLensDirective`
- `internal/loomcli#TestWire_ReviewKeysReachTheEnv`
- `internal/loomcli#seedLoomConfigWithKeys`
- `internal/configreg#TestLayerRunSettings`
- `internal/configreg#TestLookup`
- `internal/configcli#TestPrintResolved`
- `internal/configcli#TestConfigOverRealHub`
- `internal/hubreconcile_test#loomFormerUnits`
- `internal/hubreconcile_test#preMigrationBase`
- `contracts/specs/llm-model-spec.md`

**Uses:**
- `internal/loomrecipe#Rows`
- `internal/configreg/doc.go`
- `internal/boardcli/doc.go`
- `internal/reviewbudget/doc.go`

**Intent:** Split the config (D12): `loom.yaml` keeps what every loom recipe shares, and a new per-worktree module `tapestry`, file `tapestry.yaml`, takes what only `tapestry` has.

The module lives in `internal/loomengine` beside loom's, since it shares loom's review-budget family and model resolution and its load reads `loom.yaml`'s run-wide keys: `tapestry.yaml` is its embedded template (`//go:embed`), and `TapestryConfigTemplate`, `TapestryConfigOpenMaps`, `TapestryConfigSpec` (name `tapestry`, not hub-wide), `LoadTapestryConfig` and `ValidateTapestryLocalValues` mirror loom's five.

Keys moved from `loom.yaml` to `tapestry.yaml`, with their template values and comments: `plan`, `plan_timeout_min`, `implementer`, `plan_lenses`, `plan_reviewer_model`, `plan_fixer_model`, `plan_reviewer_fan`, `plan_fixer_fan`, `plan_judge`, `plan_seed`, `plan_max_bounces`, `plan_hard_max_bounces` and `plan_auto_accept`. `loom.yaml` keeps `discussion_*`, `code_*`, `review_*`, `friction*`, `reviewer_model`, `fixer_model`, `judge`, `seed`, `driver` and `step_idle_timeout_min`; `darn.yaml` is untouched here.
- `TapestryConfigSpec` classifies each moved key as loom did: `plan_lenses`, `plan_reviewer_fan`, `plan_fixer_fan` and `plan_auto_accept` guard, the rest none; the plan perch's budget keys come from `reviewbudget.Categories([]string{"plan"})` minus its three run-wide keys, which stay loom's. `ConfigSpec` and `ConfigOpenMaps` of loom drop the moved keys, and `reviewPerches` becomes discussion and code, so loom's budget categories, `LoadConfig`'s `CheckResolved` and `ValidateLocalValues` cover those two perches; `fanKeys` and `segmentModelKeys` drop their plan entries.
- `LoadTapestryConfig` loads `tapestry.yaml` strictly through `configengine.Load`, validates `plan` and each non-empty `plan_judge`, `plan_seed`, `plan_reviewer_model` and `plan_fixer_model` entry through `modelspec.Parse`, the fan keys through `lenses.ValidateFanKeys`, and a negative `plan_timeout_min`, each refusal naming `tapestry config key %q`. It loads `loom.yaml`'s run-wide `review_max_bounces`, `review_hard_max_bounces` and `review_amend_max_bounces` through `configengine.Load(dirs, ConfigSpec())` and runs `reviewbudget.CheckResolved` over the plan perch with `TapestryConfigSpec()` as its spec and `RunWideSpec` set to `ConfigSpec()`, so a `plan_hard_max_bounces` below the budget it resolves (its own, or `review_max_bounces` when `plan_max_bounces` is null) is refused as `LoadConfig` refuses one today. The fallbacks keep across modules: `plan_max_bounces: null` takes `loom.review_max_bounces`, `plan_hard_max_bounces: 0` takes `loom.review_hard_max_bounces`, an empty `plan_reviewer_model`/`plan_fixer_model` takes `loom.reviewer_model`/`fixer_model`, and an empty `plan_judge`/`plan_seed` takes `loom.judge`/`seed`.
- `reviewbudget.Values` gains `RunWideSpec configengine.Spec`, the spec of the module that holds `review_max_bounces`, `review_hard_max_bounces` and `review_amend_max_bounces`, its zero value meaning the spec argument holds them. `CheckResolved` routes each of those three keys through `RunWideSpec` and every perch key through the spec argument, and each refusal's `<module> config key` prefix, and each route of the hard-below-soft refusal, names the module that owns the key it reports: a `plan_hard_max_bounces` below a budget inherited from `review_max_bounces` names `lym config tapestry` to raise the hard cap and `lym config loom` to lower the soft cap. Loom's `LoadConfig` leaves `RunWideSpec` zero, so its refusals keep their text.
- `Config` keeps its plan fields with the tag `yaml:"-"`, and `LoadConfig` never reads `tapestry.yaml`, so a fault or an absent base of `tapestry.yaml` refuses no darn or pawl run and no recipe without a Plan-Review segment. A new method `Config.WithTapestry(TapestryConfig) Config` copies the plan fields in, and the callers that serve the Plan-Review segment call it: `wire` and `reviewBudget` load `LoadTapestryConfig` and apply it when `loomrecipe.Rows(c.recipe)` holds a row of segment `Plan-Review` (a row read, not a name branch), `reviewBudget`'s lightweight path degrading to the tapestry template with a warning as it does for loom. `Config.Caps(PerchPlan)`, `ResolveReview`, `ResolveJudge` and `PlanTable` then read the moved values unchanged; with no tapestry fields applied, the plan perch's values are the shared fallbacks no recipe without that segment reads. `TapestryConfig` holds the fields with their yaml tags. `loomSegmentModelSource("plan", …)` names module `tapestry` in the keys it reports, so a plan model refusal names `tapestry.yaml`.
- `configreg.Modules` registers `tapestry` (template, open maps, `CheckValues: ValidateTapestryLocalValues`, `CheckLoad` over `LoadTapestryConfig`, `Spec`), in alphabetical order, per-worktree.
- `configreg.KeyMoves` declares each moved key `loom` → `tapestry`, same key, so the reconcile moves a value a hub's base `loom.yaml` tuned into a sparse `tapestry.yaml` (PATTERN-sparse-hub-base), and a profile block or `run_settings` naming `loom.plan_max_bounces` is translated when read by `moveRunSettings`, nothing rewritten.
- `configsync.formerLoomValues` drops `plan_lenses`, `plan_reviewer_model`, `plan_reviewer_fan` and `plan_fixer_fan`, which `formerTemplateValues` lists under `tapestry` with the same former values.
- `tapestry-recipe.yaml`'s `module.key` references to moved keys name `tapestry`: `tapestry.plan_reviewer_fan`, `tapestry.plan_fixer_fan`, `tapestry.plan_auto_accept` and `tapestry.implementer`, and its header's "loom.yaml key" wording names `tapestry.yaml` for the plan perch.
- `loomSegmentSettings` gives Plan-Review `ConfigFile: "tapestry.yaml"`; `perchHardCap`, for the plan perch, loads `LoadTapestryConfig` and applies `Config.WithTapestry` before reading `Caps`, so the cap it returns for Plan-Bouncer is `tapestry.yaml`'s, its doc naming `tapestry.yaml` as the plan perch's cap source, and routes the hard-cap way forward through the module that owns `caps.HardKey`: `TapestryConfigSpec()` when it is `reviewbudget.HardKey("plan")`, and `ConfigSpec()` otherwise, since the template's `plan_hard_max_bounces: 0` resolves to loom's `review_hard_max_bounces`; `reviewBudget`'s template fallback also unmarshals the tapestry template into the plan fields.
- Readers of the moved keys outside the module follow, each naming module `tapestry` for a moved key:
  - `TestDispatchLayers`' local-layer row sets `plan_reviewer_model` on module `tapestry`, seeds the board with the tapestry template, and reads the local `tapestry.yaml`.
  - `engineMinimalConfig` sets `implementer_setting: tapestry.implementer`, and its doc comment says `tapestry.implementer`.
  - `implementerEntry`'s three way-forward texts give `tapestry.implementer` as the example key, and `TestImplementerEntry_RefusesAnUnusableSetting` asserts that text and names `tapestry.implementer` in its fixtures.
  - `internal/shedrecipe/doc.go`'s Implementer paragraph names `tapestry.implementer` as the shipped recipe's setting, and `planWriteEntry`'s comment names `tapestry.yaml`'s `plan_lenses`.
  - `TestSettingLookup` looks up and writes `plan_auto_accept` on module `tapestry`; `TestFullEnv_FillsEverySeam` reads `TemplateSetting("tapestry", "plan_auto_accept")`; `TestModules_LocalLayerRefusals`' `plan_lenses` and `plan_reviewer_fan` rows run against module `tapestry`.
  - battencli's `TestPrepareRunSettings` wants `plan_max_bounces` in the `tapestry` document and `code_max_bounces` in the `loom` one.
  - `TestBuild_EntryGate` and `TestBuild_Errors` use `tapestry.plan_auto_accept` and `lym config tapestry` text, and `TestReviewBudget_NegativeRefused` gives Plan-Review `ConfigFile: "tapestry.yaml"`.
  - `internal/configcli/doc.go`'s `plan_reviewer_model` example and `boardcli.Command`'s `run_settings` help example name module `tapestry`.
  - `twillLenses` takes the module beside the key, and its errors name `tapestry config key` for `plan_lenses` and `loom config key` for `discussion_lenses`; `planLensDirective` passes `tapestry` and `discussionLensDirective` passes `loom`.
  - `TestConfigTemplate_ContainsEveryConfigYAMLTag` skips a `Config` field tagged `-` and checks that every `TapestryConfig` field's tag is a key of `TapestryConfigTemplate()`; `TestConfigTemplate_ImplementerDefaultsToWebsterAndAcceptsALocalValue` reads `implementer` from `TapestryConfigTemplate()` and `TapestryConfigSpec()`.
  - `seedLoomConfigWithKeys` writes each key into `loom.yaml` or `tapestry.yaml`, whichever module's template holds it, each file from its embedded template, and fails the test on a key neither holds. `TestWire_ReviewKeysReachTheEnv` expects the Plan-Review segment's `ConfigFile` `tapestry.yaml` and its keys under module `tapestry`, and reads `env.Setting("tapestry", "plan_reviewer_fan")`.
  - configreg's `TestLayerRunSettings` expects its `plan_max_bounces` rows' document under `tapestry`, and `TestLookup` lists `plan_reviewer_model`, `plan_fixer_model`, `plan_reviewer_fan` and `plan_fixer_fan` under tapestry's open maps in place of loom's.
  - configcli's `TestPrintResolved` expects `plan_timeout_min` in the tapestry listing, and `TestConfigOverRealHub` keeps planting `plan_reviewer_fan` and `plan_fan` in the board `loom.yaml`, where a hub holds them, and expects after reconcile `plan_reviewer_fan: review-plan` in the board `tapestry.yaml` through loom's `plan_fan` rename and the `KeyMoves` move, and neither key in `loom.yaml`.
  - The hubreconcile test's `loomFormerUnits` drops `plan_lenses`, `plan_reviewer_model`, `plan_reviewer_fan` and `plan_fixer_fan`, and `preMigrationBase` gains a `tapestry` case holding them at their former values.
  - `contracts/specs/llm-model-spec.md`'s loom.yaml paragraph drops `plan_judge` and `plan_seed`, and a tapestry.yaml sentence names them, an empty one still taking loom's `judge` or `seed`.
  - `TestMigrateConfig`'s "migrated layers load with the advisors each layer's migration left" subtest drops its `PlanLenses == "twill-plan"` conditions and their error-message operands, since `LoadConfig` leaves the plan fields at their zero value and only warns on the migrated `plan_lenses` key; the table rows above that subtest keep pinning the `plan_producer` to `plan_lenses` rewrite.

Tests: `TestLoadTapestryConfig` (tier0) loads a hub where `tapestry.plan_max_bounces: null` takes `loom.review_max_bounces`, an empty `plan_reviewer_model` reports the shared list, `plan_hard_max_bounces` below the budget inherited from `loom.review_max_bounces` is refused naming the key, and a bad `plan` model-spec is refused; an empty `plan_judge` and `plan_seed` leave `ResolveJudge` of the applied config giving the plan segment `loom.judge`'s model. `TestWire_DefaultConfig` asserts a tapestry seed's Plan-Review segment settings carry `ConfigFile: tapestry.yaml`; `TestWire_DarnRecipe` asserts a darn seed wires with no `tapestry.yaml` in the hub. `implementer-spec.md` names `tapestry.yaml`'s `implementer` key. `TestLoadConfig_Loads` drops its plan-field expectations, since `LoadConfig` leaves the plan fields of `Config` at their zero value, and `TestLoadTapestryConfig` asserts the plan fields `Config.WithTapestry` applies come from `tapestry.yaml`; `TestLoadConfig_Refuses` drops its plan-perch rows to `TestLoadTapestryConfig`. `TestResolveReview` asserts `PlanKeys` names module tapestry. `TestModules_EveryTemplateKeyClassified` adds the tapestry key list and drops the moved keys from loom's. `TestPrepareRunSettings` gains a row whose `loom.plan_max_bounces` lands in the `tapestry` document. `TestReconcile_MovesKeysAcrossModules` gains a row where a base `loom.yaml` tuning `plan_max_bounces: 5` leaves it in `tapestry.yaml` after reconcile. `TestShippedRecipes_FanSettingsAgreeWithinSegment`'s Plan-Review row names the `tapestry.` keys. `TestPerchHardCap`'s Plan-Bouncer row seeds `plan_hard_max_bounces` into the worktree's `tapestry.yaml` in place of `loom.yaml`, at a value its resolved budget admits and that differs from loom's default, asserting that value is the cap `perchHardCap` returns and the raise names `lym config tapestry`; a second Plan-Bouncer row seeds no `plan_hard_max_bounces` and a local `loom.yaml` `review_hard_max_bounces`, asserting that value is the cap and the raise names `lym config loom --layer local --set review_hard_max_bounces=<n>` and never `lym config tapestry`. `TestCheckResolved` gains a row where the plan perch inherits a run-wide soft cap above its own hard cap, with `RunWideSpec` set, asserting the soft-cap route names `lym config loom` and the hard-cap route `lym config tapestry`. A second `TestCheckResolved` row, with `RunWideSpec` set, gives the plan perch its own `plan_max_bounces` above the run-wide `review_hard_max_bounces` and `plan_hard_max_bounces: 0`, asserting the refusal's prefix reads `loom config key "review_hard_max_bounces"`, the raise-the-hard-cap route names `lym config loom` and the lower-the-soft-cap route for `plan_max_bounces` names `lym config tapestry`. `docs/overview.md`'s config module table and reconcile paragraph name `tapestry.yaml`.

**ImpactSummary:** Thirteen keys move module; loom's loader fills them from the new module so its readers keep their signatures, and recipe setting references, segment settings, reconcile moves and run-settings translation follow. `reviewbudget`'s resolved-budget check learns which module holds the run-wide keys, and `twillLenses` takes the module its errors name.

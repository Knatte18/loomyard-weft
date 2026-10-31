# Card 6 — tapestry-config-module

**Create:**
- `internal/loomengine/tapestry.yaml`
- `plan:internal/loomengine#TapestryConfig` -> `type TapestryConfig struct`
- `plan:internal/loomengine#TapestryConfigTemplate` -> `func TapestryConfigTemplate() string`
- `plan:internal/loomengine#TapestryConfigOpenMaps` -> `func TapestryConfigOpenMaps() []string`
- `plan:internal/loomengine#TapestryConfigSpec` -> `func TapestryConfigSpec() configengine.Spec`
- `plan:internal/loomengine#LoadTapestryConfig` -> `func LoadTapestryConfig(dirs configengine.Dirs, module string) (TapestryConfig, error)`
- `plan:internal/loomengine#ValidateTapestryLocalValues` -> `func ValidateTapestryLocalValues(document []byte) error`
- `plan:internal/loomengine#Config.WithTapestry` -> `func (c Config) WithTapestry(t TapestryConfig) Config`
- `plan:internal/loomengine#TestLoadTapestryConfig` -> `func TestLoadTapestryConfig(t *testing.T)`

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

**Uses:**
- `internal/loomengine#TestConfigTemplate_ContainsEveryConfigYAMLTag`
- `internal/loomrecipe#Rows`
- `internal/configreg/doc.go`

**Intent:** Split the config (D12): `loom.yaml` keeps what every loom recipe shares, and a new per-worktree module `tapestry`, file `tapestry.yaml`, takes what only `tapestry` has.

The module lives in `internal/loomengine` beside loom's, since it shares loom's review-budget family and model resolution and its load reads `loom.yaml`'s run-wide keys: `tapestry.yaml` is its embedded template (`//go:embed`), and `TapestryConfigTemplate`, `TapestryConfigOpenMaps`, `TapestryConfigSpec` (name `tapestry`, not hub-wide), `LoadTapestryConfig` and `ValidateTapestryLocalValues` mirror loom's five.

Keys moved from `loom.yaml` to `tapestry.yaml`, with their template values and comments: `plan`, `plan_timeout_min`, `implementer`, `plan_lenses`, `plan_reviewer_model`, `plan_fixer_model`, `plan_reviewer_fan`, `plan_fixer_fan`, `plan_judge`, `plan_seed`, `plan_max_bounces`, `plan_hard_max_bounces` and `plan_auto_accept`. `loom.yaml` keeps `discussion_*`, `code_*`, `review_*`, `friction*`, `reviewer_model`, `fixer_model`, `judge`, `seed`, `driver` and `step_idle_timeout_min`; `darn.yaml` is untouched here.
- `TapestryConfigSpec` classifies each moved key as loom did: `plan_lenses`, `plan_reviewer_fan`, `plan_fixer_fan` and `plan_auto_accept` guard, the rest none; the plan perch's budget keys come from `reviewbudget.Categories([]string{"plan"})` minus its three run-wide keys, which stay loom's. `ConfigSpec` and `ConfigOpenMaps` of loom drop the moved keys, and `reviewPerches` becomes discussion and code, so loom's budget categories, `LoadConfig`'s `CheckResolved` and `ValidateLocalValues` cover those two perches; `fanKeys` and `segmentModelKeys` drop their plan entries.
- `LoadTapestryConfig` loads `tapestry.yaml` strictly through `configengine.Load`, validates `plan` and each non-empty `plan_judge`, `plan_seed`, `plan_reviewer_model` and `plan_fixer_model` entry through `modelspec.Parse`, the fan keys through `lenses.ValidateFanKeys`, and a negative `plan_timeout_min`, each refusal naming `tapestry config key %q`. It loads `loom.yaml`'s run-wide `review_max_bounces`, `review_hard_max_bounces` and `review_amend_max_bounces` through `configengine.Load(dirs, ConfigSpec())` and runs `reviewbudget.CheckResolved` over the plan perch, so a `plan_hard_max_bounces` below the budget it resolves (its own, or `review_max_bounces` when `plan_max_bounces` is null) is refused as `LoadConfig` refuses one today. The fallbacks keep across modules: `plan_max_bounces: null` takes `loom.review_max_bounces`, `plan_hard_max_bounces: 0` takes `loom.review_hard_max_bounces`, an empty `plan_reviewer_model`/`plan_fixer_model` takes `loom.reviewer_model`/`fixer_model`, and an empty `plan_judge`/`plan_seed` takes `loom.judge`/`seed`.
- `Config` keeps its plan fields with the tag `yaml:"-"`, and `LoadConfig` never reads `tapestry.yaml`, so a fault or an absent base of `tapestry.yaml` refuses no darn or pawl run and no recipe without a Plan-Review segment. A new method `Config.WithTapestry(TapestryConfig) Config` copies the plan fields in, and the callers that serve the Plan-Review segment call it: `wire` and `reviewBudget` load `LoadTapestryConfig` and apply it when `loomrecipe.Rows(c.recipe)` holds a row of segment `Plan-Review` (a row read, not a name branch), `reviewBudget`'s lightweight path degrading to the tapestry template with a warning as it does for loom. `Config.Caps(PerchPlan)`, `ResolveReview`, `ResolveJudge` and `PlanTable` then read the moved values unchanged; with no tapestry fields applied, the plan perch's values are the shared fallbacks no recipe without that segment reads. `TapestryConfig` holds the fields with their yaml tags. `loomSegmentModelSource("plan", …)` names module `tapestry` in the keys it reports, so a plan model refusal names `tapestry.yaml`.
- `configreg.Modules` registers `tapestry` (template, open maps, `CheckValues: ValidateTapestryLocalValues`, `CheckLoad` over `LoadTapestryConfig`, `Spec`), in alphabetical order, per-worktree.
- `configreg.KeyMoves` declares each moved key `loom` → `tapestry`, same key, so the reconcile moves a value a hub's base `loom.yaml` tuned into a sparse `tapestry.yaml` (PATTERN-sparse-hub-base), and a profile block or `run_settings` naming `loom.plan_max_bounces` is translated when read by `moveRunSettings`, nothing rewritten.
- `configsync.formerLoomValues` drops `plan_lenses`, `plan_reviewer_model`, `plan_reviewer_fan` and `plan_fixer_fan`, which `formerTemplateValues` lists under `tapestry` with the same former values.
- `tapestry-recipe.yaml`'s `module.key` references to moved keys name `tapestry`: `tapestry.plan_reviewer_fan`, `tapestry.plan_fixer_fan`, `tapestry.plan_auto_accept` and `tapestry.implementer`, and its header's "loom.yaml key" wording names `tapestry.yaml` for the plan perch.
- `loomSegmentSettings` gives Plan-Review `ConfigFile: "tapestry.yaml"`; `perchHardCap` routes the plan perch's hard-cap way forward through `TapestryConfigSpec()`; `reviewBudget`'s template fallback also unmarshals the tapestry template into the plan fields.

Tests: `TestLoadTapestryConfig` (tier0) loads a hub where `tapestry.plan_max_bounces: null` takes `loom.review_max_bounces`, an empty `plan_reviewer_model` reports the shared list, `plan_hard_max_bounces` below the budget inherited from `loom.review_max_bounces` is refused naming the key, and a bad `plan` model-spec is refused; an empty `plan_judge` and `plan_seed` leave `ResolveJudge` of the applied config giving the plan segment `loom.judge`'s model. `TestWire_DefaultConfig` asserts a tapestry seed's Plan-Review segment settings carry `ConfigFile: tapestry.yaml`; `TestWire_DarnRecipe` asserts a darn seed wires with no `tapestry.yaml` in the hub. `implementer-spec.md` names `tapestry.yaml`'s `implementer` key. `TestLoadConfig_Loads` asserts the plan fields of `Config` come from `tapestry.yaml`; `TestLoadConfig_Refuses` drops its plan-perch rows to `TestLoadTapestryConfig`. `TestResolveReview` asserts `PlanKeys` names module tapestry. `TestModules_EveryTemplateKeyClassified` adds the tapestry key list and drops the moved keys from loom's. `TestPrepareRunSettings` gains a row whose `loom.plan_max_bounces` lands in the `tapestry` document. `TestReconcile_MovesKeysAcrossModules` gains a row where a base `loom.yaml` tuning `plan_max_bounces: 5` leaves it in `tapestry.yaml` after reconcile. `TestShippedRecipes_FanSettingsAgreeWithinSegment`'s Plan-Review row names the `tapestry.` keys. `docs/overview.md`'s config module table and reconcile paragraph name `tapestry.yaml`.

**ImpactSummary:** Thirteen keys move module; loom's loader fills them from the new module so its readers keep their signatures, and recipe setting references, segment settings, reconcile moves and run-settings translation follow.

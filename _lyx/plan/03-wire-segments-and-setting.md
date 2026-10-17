# Card 3 — wire-segments-and-setting

**Create:**
- `plan:internal/shedrecipe#SegmentSettings` -> `type SegmentSettings struct`
- `plan:internal/loomcli#loomSegmentPerches` -> `var loomSegmentPerches map[string]loomengine.Perch`
- `plan:internal/loomcli#darnReviewSegment` -> `const darnReviewSegment = "Darn-Review"`
- `plan:internal/loomcli#loomSegmentSettings` -> `func loomSegmentSettings(cfg loomengine.Config, review loomengine.ReviewSettings, judge loomengine.JudgeSettings) map[string]shedrecipe.SegmentSettings`
- `plan:internal/testkit/envkit#TemplateSetting` -> `func TemplateSetting(module, key string) (any, error)`

**Edit:**
- `internal/shedrecipe#Env`
- `internal/loomcli#loomCLI.wire`
- `internal/loomcli#loomCLI.wireDarn` -> `func (c *loomCLI) wireDarn(location *lyxcwd.Location, stencilsDir, specsDir, frictionDir string, landingCfg landingshed.Config, registry modelspec.Registry, shared burlerengine.RoundModels, bouncer shedadapters.BouncerModels) error`
- `internal/loomcli#wireDarnReview` -> `func wireDarnReview(env *shedrecipe.Env, cfg darnengine.Config, shared burlerengine.RoundModels, bouncer shedadapters.BouncerModels, registry modelspec.Registry) error`
- `internal/loomcli#TestWire_ReviewKeysReachTheEnv`
- `internal/loomcli#TestWire_DarnRecipe`
- `internal/testkit/envkit#FullEnv`
- `internal/testkit/envkit#nilLegal`
- `internal/testkit/envkit#TestFullEnv_FillsEverySeam`

**Uses:**
- `internal/configreg#Lookup`
- `internal/loomcli#seedLoomConfigWithKeys`
- `internal/loomcli#seedDarnHubConfig`

**Intent:** Add the two new `Env` fields and fill them from the loom and darn wiring and the shared test kit, beside the per-row maps they replace; no entry reads them yet (card 4), so behavior is unchanged.

`SegmentSettings` (D1) holds one review segment's resolved values, each field documented:
- `Models burlerengine.RoundModels`, the reviewer and fixer round models, already fallen back to the shared `reviewer_model` and `fixer_model`;
- `ReviewKey` and `FixKey shuttleengine.ModelKey`, the config keys those models came from;
- `Bouncer shedadapters.BouncerModels`, the Bouncer's seed and judge pair;
- `MaxBounces int`, the resolved bounce budget, 0 meaning zero rounds, and `MaxBouncesKey string`, the key it resolved from;
- `HardMaxBounces int`, the resolved hard cap as a round number, 0 meaning none, and `HardMaxBouncesKey string`;
- `ConfigFile string`, the file both keys live in (`loom.yaml` or `darn.yaml`).

`Env` gains, each with a doc comment:
- `Segments map[string]SegmentSettings`, keyed by the recipe's `segment:` value, read by the Bouncer, BurlerReview and BurlerFix entries and by `internal/loomrecipe`; its comment states that it is legal on `Env` because a segment's three rows share one entry;
- `Setting func(module, key string) (any, error)`, the told setting lookup a row reaches by naming `module.key`, returning the module's resolved value from the pair's config stack; nil is legal for a caller whose recipe names no setting.
`Env`'s opening comment ("never a value that differs between two rows") gains that `Segments` is keyed by segment and `Setting` by a row's own reference.

Loom wiring (`loomCLI.wire`):
- `loomSegmentPerches` maps `Discussion-Review`, `Plan-Review`, `Webster-Review` and `Merge-Review` to `loomengine.PerchDiscussion`, `PerchPlan`, `PerchWebster` and `PerchMerge`; its comment names it the one table a new loom perch adds a row to.
- `loomSegmentSettings` builds one entry per table row: `Models` and the keys from the perch's `ReviewSettings` field (`Discussion`/`DiscussionKeys` and so on), `Bouncer` from the perch's `JudgeSettings` field with Merge taking `Webster` as today, and `MaxBounces`, `MaxBouncesKey`, `HardMaxBounces`, `HardMaxBouncesKey` from `cfg.Caps(perch)`'s `Soft`, `SoftKey`, `Hard` and `HardKey`, with `ConfigFile` `loom.yaml`; the budget and hard cap resolve through `reviewbudget` via `Caps`, the resolution the existing `perchMaxBounces` and `perchHardMaxBounces` give.
- `wire` sets `Segments: loomSegmentSettings(loomCfg, reviewSettings, judgeSettings)` and `Setting: configreg.SettingLookup(fabricengine.ConfigDirs(location))`, the same `Dirs` the module loaders read, one lookup per wiring and so per recipe build; it passes `reviewSettings.Models` and `shedadapters.BouncerModels(judgeSettings.Webster)` to `wireDarn`.

Darn wiring:
- `wireDarn` takes the shared models and the Darn Bouncer pair and hands them to `wireDarnReview`.
- `wireDarnReview` resolves the models with `loomengine.ResolveDarnReview(shared, cfg, registry)`, keeping today's per-row map writes, and adds `env.Segments[darnReviewSegment]`: those models and keys, `Bouncer: bouncer`, `MaxBounces: cfg.ReviewMaxBounces` with `MaxBouncesKey: reviewbudget.RunSoftKey`, `HardMaxBounces: cfg.ReviewHardMaxBounces` with `HardMaxBouncesKey: reviewbudget.RunHardKey`, and `ConfigFile: "darn.yaml"`.
  Each empty darn model key falls back to the shared lists it is handed, as `ResolveDarnReview` already does.

Test kit (`internal/testkit/envkit`):
- `TemplateSetting` answers any registered module's template key from that module's embedded template (`configreg.Lookup`, the template decoded with `yaml.v3`), and errors naming the module for an unregistered module and naming the key for a key the template lacks, so a new recipe `_setting` key needs no kit change.
- `FullEnv` sets `Setting: TemplateSetting` and fills `Segments` with an entry for every review segment of the embedded recipes (`Discussion-Review`, `Plan-Review`, `Webster-Review`, `Merge-Review`, `Darn-Review`), each with empty models, `MaxBounces` equal to the kit's current run-wide budget (3, and 1 for `Darn-Review`, today's `DarnReviewMaxBounces`), `MaxBouncesKey` `review_max_bounces` and `ConfigFile` `loom.yaml` or `darn.yaml`, so packages built on the kit keep today's budgets.
- `nilLegal` gains `Segments` and `Setting`, whose nil is legal for a recipe with no review segment or no named setting, so `battencli`'s wiring test, which fills neither, needs no change.
- `TestFullEnv_FillsEverySeam` gains subtests: `TemplateSetting("loom", "plan_auto_accept")` answers `false`, an unregistered module and an unknown key each error naming the module or key, and `FullEnv(t).Segments` holds an entry for every review segment of the embedded loom and darn recipes (rows parsed with `shedbuild.Parse` over `recipes.LoomRecipe` and `recipes.DarnRecipe`, a segment counting as a review segment when it holds a `Bouncer` row).

Tests:
- `TestWire_ReviewKeysReachTheEnv` asserts, for its seeded `loom.yaml`, that `c.env.Segments` equals the expected map: each loom segment's models, keys and Bouncer pair as the existing per-row assertions expect, and each budget and hard cap with its key and `loom.yaml`, including a perch whose own `<perch>_max_bounces` is set (key `<perch>_max_bounces`) and one that falls back (key `review_max_bounces`); and that `c.env.Setting("loom", "plan_reviewer_fan")` answers the seeded value.
- `TestWire_DarnRecipe` asserts `c.env.Segments["Darn-Review"]` for its template and local cases: models falling back to the loom shared lists when the darn model keys are empty, the darn keys when set, the Webster judge pair, and budget, hard cap, keys and `darn.yaml`.

**ImpactSummary:** Two new Env fields filled by loomcli's wire and wireDarn, the kit's FullEnv and nilLegal; wireDarn and wireDarnReview each gain two parameters with one caller each; no entry reads the new fields yet.

**Commit:** 3: shedrecipe, loomcli: add Env.Segments and Env.Setting and fill them from the wiring

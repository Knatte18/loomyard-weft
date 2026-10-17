# Card 5 — wire-segments-and-setting

**Create:**
- `internal/loomcli#loomSegmentPerches`
- `internal/loomcli#darnReviewSegment`
- `internal/loomcli#loomSegmentSettings`

**Edit:**
- `internal/loomcli#loomCLI.wire`
- `internal/loomcli#loomCLI.wireDarn` -> `func (c *loomCLI) wireDarn(location *lyxcwd.Location, stencilsDir, specsDir, frictionDir string, landingCfg landingshed.Config, registry modelspec.Registry, shared burlerengine.RoundModels, bouncer shedadapters.BouncerModels) error`
- `internal/loomcli#wireDarnReview` -> `func wireDarnReview(env *shedrecipe.Env, cfg darnengine.Config, shared burlerengine.RoundModels, bouncer shedadapters.BouncerModels, registry modelspec.Registry) error`
- `internal/loomcli#TestWire_ReviewKeysReachTheEnv`
- `internal/loomcli#TestWire_DarnRecipe`

**Uses:**
- `internal/configreg#SettingLookup`
- `internal/shedrecipe#SegmentSettings`
- `internal/loomcli#seedLoomConfigWithKeys`
- `internal/loomcli#seedDarnHubConfig`

**Intent:** Fill card 3's `Env.Segments` and `Env.Setting` from the loom and darn wiring, beside the per-row maps they replace; no entry reads them yet (card 6), so behavior is unchanged.

Loom wiring (`loomCLI.wire`):
- `loomSegmentPerches` maps `Discussion-Review`, `Plan-Review`, `Webster-Review` and `Merge-Review` to `loomengine.PerchDiscussion`, `PerchPlan`, `PerchWebster` and `PerchMerge`; its comment names it the segment-to-perch table the segment settings are read through, and says a new loom perch also adds a row to `loomPerches`, which the hard-cap lookup keeps.
- `loomSegmentSettings` builds one entry per table row: `Models` and the keys from the perch's `ReviewSettings` field (`Discussion`/`DiscussionKeys` and so on), `Bouncer` from the perch's `JudgeSettings` field with Merge taking `Webster` as today, and `MaxBounces`, `MaxBouncesKey`, `HardMaxBounces`, `HardMaxBouncesKey` from `cfg.Caps(perch)`'s `Soft`, `SoftKey`, `Hard` and `HardKey`, with `ConfigFile` `loom.yaml`; the budget and hard cap resolve through `reviewbudget` via `Caps`, the resolution the existing `perchMaxBounces` and `perchHardMaxBounces` give.
- `wire` sets `Segments: loomSegmentSettings(loomCfg, reviewSettings, judgeSettings)` and `Setting: configreg.SettingLookup(fabricengine.ConfigDirs(location))`, the same `Dirs` the module loaders read, one lookup per wiring and so per recipe build; it passes `reviewSettings.Models` and `shedadapters.BouncerModels(judgeSettings.Webster)` to `wireDarn`.

Darn wiring:
- `wireDarn` takes the shared models and the Darn Bouncer pair and hands them to `wireDarnReview`.
- `wireDarnReview` resolves the models with `loomengine.ResolveDarnReview(shared, cfg, registry)`, keeping today's per-row map writes, and adds `env.Segments[darnReviewSegment]`: those models and keys, `Bouncer: bouncer`, `MaxBounces: cfg.ReviewMaxBounces` with `MaxBouncesKey: reviewbudget.RunSoftKey`, `HardMaxBounces: cfg.ReviewHardMaxBounces` with `HardMaxBouncesKey: reviewbudget.RunHardKey`, and `ConfigFile: "darn.yaml"`.
  Each empty darn model key falls back to the shared lists it is handed, as `ResolveDarnReview` already does.

Tests:
- `TestWire_ReviewKeysReachTheEnv` asserts, for both of its wirings, that `env.Segments` equals the expected map: each loom segment's models, keys and Bouncer pair as the existing per-row assertions expect, and each budget and hard cap with its key and `loom.yaml`.
  For the first wiring `c` that includes a perch whose own `<perch>_max_bounces` is set (key `<perch>_max_bounces`) and one that falls back (key `review_max_bounces`).
  For the second wiring `fanned` it includes the segments' own model keys its seed sets: Discussion-Review's `ReviewKey` `discussion_reviewer_model` with its `Models`, and Webster-Review's `FixKey` `webster_fixer_model` with its `Models`.
  It also asserts that `fanned.env.Setting("loom", "plan_reviewer_fan")` answers `"full"`, the value `fanned`'s seed sets.
- `TestWire_DarnRecipe` asserts `c.env.Segments["Darn-Review"]` for its template and local cases: models falling back to the loom shared lists when the darn model keys are empty, the darn keys when set, the Webster judge pair, and budget, hard cap, keys and `darn.yaml`.
  In its "set fans and models" case, whose seed sets `darn_fixer_fan: fix-code`, it also asserts that `c.env.Setting("darn", "darn_fixer_fan")` answers `"fix-code"`.

**ImpactSummary:** loomcli's wire and wireDarn fill the two Env fields card 3 added; wireDarn and wireDarnReview each gain two parameters with one caller each; no entry reads the new fields yet.

**Commit:** 5: loomcli: fill Env.Segments and Env.Setting from the loom and darn wiring

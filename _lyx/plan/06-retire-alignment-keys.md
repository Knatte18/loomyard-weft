# Card 6 — retire-alignment-keys

**Edit:**
- `internal/loomengine/config.go`
- `internal/loomengine/template.yaml`
- `internal/loomengine#TestLoadConfig_Loads`
- `internal/loomengine#TestLoadConfig_Refuses`
- `internal/loomengine#TestMigrateConfig_RetiredKeys`
- `internal/loomengine#templateConfig`
- `internal/configreg#TestModules_EveryTemplateKeyClassified`
- `internal/configreg#TestPrepareRunSettings`
- `internal/loomcli#seedLoomConfigWithFriction`
- `internal/loomcli#seedLoomConfigWithInteractive`
- `internal/webstercli#seedLoomConfigWithFriction`
- `pattern/PATTERN-shuttle-provider-seam.md`

**Delete:**
- `internal/loomengine#alignmentDefaultModelSpec`
- `internal/loomengine#alignmentDefaultTimeoutMin`
- `internal/loomengine#AlignmentSettings`
- `internal/loomengine#ResolveAlignment`
- `internal/loomengine#TestResolveAlignment`

**Intent:** D3: `loom.yaml`'s `alignment`, `alignment_timeout_min`, `skip_alignment` and `parent_review_wait_min` leave the template and `Config`, and the one reconciler drops them.

- `template.yaml` loses the four lines.
- `config.go`: `Config` loses `Alignment`, `AlignmentTimeoutMin`, `SkipAlignment` and `ParentReviewWaitMin`; `ConfigSpec` loses their categories and the doc sentence on them; `LoadConfig` loses the alignment model-spec check, the negative-timeout refusal, the `parent_review_wait_min` floor and the timeout-below-wait coherence check; the file header loses the alignment paragraph.
  D2's parent-review paths go from the same file: `LoomParentReviewDirRel`, `LoomParentReviewDir`, `LoomParentReviewLockDir` and `parentReviewDirName`, whose last callers cards 2 to 4 removed and whose test card 5 deleted.
  Append the four keys to `retiredLoomKeys`, so `MigrateConfig` drops each with the line `<key>: retired, removed`, and say so in `MigrateConfig`'s doc beside the Merge-Review keys.
- `driver.go`: delete `ResolveAlignment`, `AlignmentSettings` and the two defaults, whose one caller was the alignment wiring card 3 deleted.
- Reconcile drops a hub or worktree file's retired key and reports it, through `MigrateConfig`, and `configreg.PrepareRunSettings` runs the same `MigrateConfig` over a run profile's or a board `run_settings` loom object, logging each line at Warn, so a profile holding a retired key loads with a Warn, as `fc75aef79` did for the `merge_*` keys (`PATTERN-config-layers`).
- Tests: `TestMigrateConfig_RetiredKeys` gains a row whose document holds all four keys beside `plan_max_bounces: 2`, wanting one `retired, removed` line per key, `plan_max_bounces` kept and the four gone; `TestLoadConfig_Loads` and `TestLoadConfig_Refuses` drop their alignment and parent-review rows and fixtures, `templateConfig` its expected fields, and `TestResolveAlignment` goes.
  `TestModules_EveryTemplateKeyClassified` drops the three guard keys from loom's expected guard list, and `TestPrepareRunSettings` replaces its three alignment rows with one row whose loom object holds `skip_alignment: true` and `plan_max_bounces: 2`, wanting no refusal and a `loom` document holding `plan_max_bounces: 2` alone, the read-time migration of a retired key; its existing "each migration line is logged at Warn" subtest already asserts the Warn every migrated line takes.
  loomcli's `seedLoomConfigWithFriction` and `seedLoomConfigWithInteractive`, and webstercli's `seedLoomConfigWithFriction`, stop writing the retired keys, since a strict load refuses them.
- `PATTERN-shuttle-provider-seam.md`'s resolver list drops the alignment settings.

**ImpactSummary:** Four config keys and the alignment resolver go; the readers are loomengine and its tests, configreg's tests and three loomcli and webstercli test seeds.

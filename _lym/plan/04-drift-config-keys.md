# Card 4 — drift-config-keys

**Edit:**
- `internal/driftengine#Config`
- `internal/driftengine#configFile`
- `internal/driftengine#ConfigSpec`
- `internal/driftengine#LoadConfig`
- `internal/driftengine/template.yaml`
- `internal/driftengine#BaselineTable`
- `internal/driftengine#VerifyTable`
- `internal/driftengine#TestLoadConfig`
- `internal/driftengine#TestSeatTables`
- `internal/driftengine#templateConfig`
- `internal/driftengine#newVerifyFixture`
- `internal/driftengine#TestDriftVerify`
- `internal/driftengine/doc.go`
- `internal/configreg#Modules`
- `internal/configreg#PrepareRunSettings`
- `internal/configreg#TestPrepareRunSettings`
- `internal/configreg#TestModules_EveryTemplateKeyClassified`
- `internal/configreg/doc.go`

**Extends:**
- `internal/configengine#Spec`
- `internal/configengine/doc.go`

**Create:**
- `internal/driftengine#DiffLines`
- `internal/driftengine#DiffCommand`
- `internal/driftengine#MigrateConfig`
- `internal/driftengine#DiffModeReinterpretation`
- `internal/driftengine#FlagDiffMode`
- `internal/configreg#refuseHubOnlyKeys`
- `internal/driftengine#TestMigrateConfig`
- `internal/driftengine#TestFlagDiffMode`

**Uses:**
- `internal/yamlengine#RenameKeys`
- `internal/driftengine#CompileKeyPatterns`
- `internal/configreg#Reinterpretation`

**Intent:** Carry D15's key changes but the removals, which wait for the code that reads `skip` and `verify_max_runs` to go (card 26).

- `template.yaml`: add `diff: lines` (`lines` or `command`, read per D2) and `key_patterns` holding D1's two defaults, a `name = value` and a `name: value` regular expression with named groups `key` and `value`; rename `baseline_model` to `expect_model`, `verify_model` to `judge_model` and `verify_interactive` to `judge_interactive`, each comment rewritten for the expect agent and the judge; `compare`'s comment says it runs only under `diff: command`.
- `Config` and `configFile`: `BaselineModel`, `VerifyModel` and `VerifyInteractive` become `ExpectModel`, `JudgeModel` and `JudgeInteractive`; add `Diff string` and `KeyPatterns []string`.
  `BaselineTable` and `VerifyTable` read the renamed fields, which leaves their behavior as it is until card 24 removes them; `TestSeatTables` and `TestLoadConfig` move to the new names, as do the test members that set or read them: `templateConfig` (`BaselineModel`, `VerifyModel`), `newVerifyFixture` (`VerifyInteractive`) and `TestDriftVerify` (`VerifyInteractive`).
- `LoadConfig` refuses, through `refuseKey`, a `diff` other than `lines` or `command`, a `key_patterns` entry `CompileKeyPatterns` refuses, and `diff: command` with no `compare` declared, naming `compare` as the key to set (D2).
- `ConfigSpec` classifies `expect_model`, `judge_model` and `judge_interactive` as `CategoryNone` and `diff` and `key_patterns` as `CategoryGuard`, so a local layer cannot set them, and fills the new `configengine.Spec` field `HubOnlyWayForward` for `diff` and `key_patterns` with `set it in the hub's drift.yaml with lym config drift --set <key>=<value> from the prime`.
- `configengine.Spec` gains `HubOnlyWayForward map[string]string`, keys a task's run settings may not carry, each with its way forward.
  `PrepareRunSettings` calls `refuseHubOnlyKeys` beside `refuseRetiredKeys`: a top-level key the module's spec maps there is refused, in name order, as a `*RunSettingsError` marked invalid, `key "<k>" of module "<m>" is set only in the hub's config; way forward: <text>`.
  Bound: the field only refuses; a key it names still loads from the hub layers.
- `MigrateConfig` renames the three keys through `yamlengine.RenameKeys` against the template, which keeps each value; `Modules` registers it as the drift module's `Migrate`, so a hub base migrates at reconcile and a task's run settings carrying an old key migrate with a Warn.
- `FlagDiffMode` rewrites nothing and flags, one line each: a set `compare` with `diff` unset (`compare` now runs only under `diff: command`, and the capture must meet D1's contract to keep the default), a `capture` or `compare` command holding `{skip_file}` (naming the edit to drop it), and `scope: partial` (partial is gone and the base runs as `full`; way forward `lym config drift --set scope=full` or `scope=off`, and `lym config reconcile` prunes `skip`).
  `Modules` adds it as a second drift `Reinterpretation` under `DiffModeReinterpretation`, applied once per hub by `hubreconcile` (`PATTERN-config-reinterpretation`).

Tests: `TestLoadConfig` gains rows for the template's `diff` and patterns, a refused `diff` value, a refused pattern, and `diff: command` without `compare` refused naming `compare`; `TestMigrateConfig` (tier0) covers the three renames keeping values and a document already holding the new key; `TestFlagDiffMode` (tier0) covers each flag, a base with none, and that the document is never rewritten.
`TestPrepareRunSettings` gains rows: `{"drift": {"diff": "command"}}` and `{"drift": {"key_patterns": []}}` refused naming the hub's drift.yaml, and `{"drift": {"verify_model": "opus"}}` migrating to `judge_model`.
`TestModules_EveryTemplateKeyClassified`'s drift local keys become `enabled`, `expect_model`, `judge_interactive`, `judge_model`, `scope`, `skip`, `verify_max_runs`, and `diff` and `key_patterns` join drift's denied keys.
The doc.go files of driftengine, configengine and configreg describe the new keys, the field and the refusal.

**ImpactSummary:** Renames three drift Config fields read by two seat builders in the package; adds one configengine Spec field read only by configreg's run-settings path; every other module is unaffected since the field is empty for them.

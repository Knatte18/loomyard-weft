# Card 5 — slack-config

**Create:**
- `internal/slackengine/template.yaml`
- `plan:internal/slackengine#ConfigTemplate` -> `func ConfigTemplate() string`
- `plan:internal/slackengine#ConfigSpec` -> `func ConfigSpec() configengine.Spec`
- `plan:internal/slackengine#Config` -> `type Config struct`
- `plan:internal/slackengine#LoadConfig` -> `func LoadConfig(dirs configengine.Dirs) (Config, error)`
- `internal/slackengine/config_test.go`
- `plan:internal/slackengine#TestLoadConfig` -> `func TestLoadConfig(t *testing.T)`

**Edit:**
- `internal/configreg#Modules`
- `internal/configreg#TestModules_SeedOnlyAndHubWideFlags`
- `internal/configreg#TestModules_EveryTemplateKeyClassified`
- `cmd/lym#configStrictnessDegradingSet`
- `pattern/PATTERN-config-layers.md`

**Extends:**
- `internal/slackengine/doc.go`

**Uses:**
- `internal/configreg/doc.go`
- `internal/configengine#LoadOrTemplate`
- `internal/hibernateconfig#ConfigSpec`

**Intent:** Add the hub-wide `slack` config module, which holds only `remind_after_min` (the record's Delivers and Exports lines; the reminders that read it are D8's, outside this part).

- `template.yaml` holds one key, `remind_after_min: 30`, with a comment saying it is the minutes between the orch's reminders while presence is `slack`, and that the channel, the operator and the token are never config: `lym slack setup` and `lym slack login` write them (D2).
- `ConfigTemplate`, `ConfigSpec` and `LoadConfig` follow `internal/hibernateconfig`: module name `slack`, `HubWide: true`, the one key classified `configengine.CategoryNone`, since its value bounds no resource and switches off no guard; `LoadConfig` degrades through `configengine.LoadOrTemplate` and refuses a value below 1 through `configengine.MarkInvalid`, naming the key and the floor, with the way forward "set `remind_after_min` to 1 or more with `lym config slack`".
  `Config{RemindAfterMin int}` is the exported loader's result.
- `configreg.Modules` registers `{Name: "slack", Template: slackengine.ConfigTemplate, HubWide: true, CheckLoad: checkLoad(slackengine.LoadConfig), Spec: slackengine.ConfigSpec}` in sort order.
- `TestModules_SeedOnlyAndHubWideFlags` adds `slack` to its hub-wide chain and its doc comment's list.
- `TestModules_EveryTemplateKeyClassified` records `slack`'s local keys as `remind_after_min`, the one key, which no category closes.
- `cmd/lym`'s `configStrictnessDegradingSet` adds `internal/slackengine`, and `pattern/PATTERN-config-layers.md`'s degrading-callers set names `slackengine` too.
- `doc.go` gains a sentence on the module.

`internal/slackengine/config_test.go` is tier0: `TestLoadConfig` rows over `t.TempDir()` hub dirs — no file reads the template's 30; a hub file of 45 reads 45; a hub file of 0 is refused naming `remind_after_min` and `lym config slack`; and the template's key set is exactly `remind_after_min`, so no `lym config slack` write can name a channel, an operator or a token, and none repoints a live relay (D2's criterion).

**ImpactSummary:** One new hub-wide module registered in configreg; three existing tests and one background list gain its name, and no other module or caller changes.

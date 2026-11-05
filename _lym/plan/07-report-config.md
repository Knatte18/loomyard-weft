# Card 7 — report-config

**Create:**
- `internal/reportengine/template.yaml`
- `plan:internal/reportengine#Config` -> `type Config struct`
- `plan:internal/reportengine#ConfigTemplate` -> `func ConfigTemplate() string`
- `plan:internal/reportengine#ConfigSpec` -> `func ConfigSpec() configengine.Spec`
- `plan:internal/reportengine#LoadConfig` -> `func LoadConfig(dirs configengine.Dirs) (Config, error)`
- `plan:internal/reportengine#TestLoadConfig` -> `func TestLoadConfig(t *testing.T)`

**Edit:**
- `internal/configreg#Modules`
- `internal/configreg#TestNames`
- `internal/reportengine#allowedImports`

**Uses:**
- `internal/configreg#TestModules_EveryTemplateKeyClassified`
- `internal/configreg/doc.go`

**Intent:** The report key D13 names, `cost_min_runs`, in a new hub-wide `report` config module, declared the way `internal/heddleengine/config.go` declares its module (`PATTERN-config-layers`: only `configengine` reads the layers, through `LoadOrTemplate`).
`template.yaml` holds one key, `cost_min_runs: 5  # runs a cell of lym report cost needs before it shows its figures instead of "too few runs"`, embedded by `ConfigTemplate`.
`Config` carries `CostMinRuns int` (`yaml:"cost_min_runs"`); `LoadConfig` refuses a value under 1, naming the key and the way forward `lym config report`.
`ConfigSpec` names the module `report`, hub-wide, its key cross-pair.
`configreg.Modules` gains `{Name: "report", Template: reportengine.ConfigTemplate, HubWide: true, CheckLoad: checkLoad(reportengine.LoadConfig), Spec: reportengine.ConfigSpec}` in name order, and `TestNames` its name.
`allowedImports` gains `internal/configengine`.
`TestLoadConfig` (tier0) has rows: the template value, a hub value, and a zero value refused naming the key.
The config fingerprint changes with the new module, as any new module's does.

**ImpactSummary:** A new config module with one key; the registry's per-module tests read it through the registry, and only card 8 reads the key.

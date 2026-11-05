# Card 7 — report-config

**Edit:**
- `internal/reportconfig/doc.go`
- `internal/reportconfig_test#TestMain`
- `internal/reportconfig/template.yaml`
- `internal/reportconfig#Config`
- `internal/reportconfig#ConfigTemplate`
- `internal/reportconfig#ConfigSpec`
- `internal/reportconfig#LoadConfig`
- `internal/reportconfig_test#TestLoadConfig`
- `internal/configreg#Modules`
- `internal/configreg#TestModules_SeedOnlyAndHubWideFlags`
- `cmd/lym#configStrictnessDegradingSet`
- `pattern/PATTERN-config-layers.md`

**Uses:**
- `internal/configreg#TestModules_EveryTemplateKeyClassified`
- `internal/configreg/doc.go`

**Intent:** The report key D13 names, `cost_min_runs`, in a new hub-wide `report` config module, declared the way `internal/heddleengine/config.go` declares its module (`PATTERN-config-layers`: only `configengine` reads the layers, through `LoadOrTemplate`).
The module sits in its own package, `internal/reportconfig`, because `internal/reportengine` already depends on `internal/configreg`, and `configreg.Modules` importing a module declared in `internal/reportengine` would be an import cycle.
`template.yaml` holds one key, `cost_min_runs: 5  # runs a cell of lym report cost needs before it shows its figures instead of "too few runs"`, embedded by `ConfigTemplate`.
`Config` carries `CostMinRuns int` (`yaml:"cost_min_runs"`); `LoadConfig` refuses a value under 1, naming the key and the way forward `lym config report`.
`ConfigSpec` names the module `report`, hub-wide, its key cross-pair.
`configreg.Modules` gains `{Name: "report", Template: reportconfig.ConfigTemplate, HubWide: true, CheckLoad: checkLoad(reportconfig.LoadConfig), Spec: reportconfig.ConfigSpec}` in name order.
`TestModules_SeedOnlyAndHubWideFlags`' `wantHubWide` literal, which names every hub-wide module, gains `report`.
`configStrictnessDegradingSet`, the pinned set of packages that load config with `LoadOrTemplate`, gains `internal/reportconfig`.
`pattern/PATTERN-config-layers.md`'s Strictness section, whose Degrading callers set lists the packages that load with `LoadOrTemplate`, gains `reportconfig`.
`testmain_test.go` is untagged, the package's `TestMain`.
`TestLoadConfig` (tier0) has rows: the template value, a hub value, and a zero value refused naming the key.
The config fingerprint changes with the new module, as any new module's does.

**ImpactSummary:** A new config module with one key; the registry's per-module tests read it through the registry, and only card 8 reads the key.

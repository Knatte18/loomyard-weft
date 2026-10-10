# Card 2 — gate-config

**Create:**
- `plan:internal/gateslot#ConfigTemplate` -> `func ConfigTemplate() string`
- `plan:internal/gateslot#Config` -> `type Config struct`
- `plan:internal/gateslot#LoadConfig` -> `func LoadConfig(baseDir string) (Config, error)`
- `plan:internal/gateslot#TemplateConfig` -> `func TemplateConfig() (Config, error)`
- `plan:internal/gateslot#Config.Limits` -> `func (c Config) Limits() Limits`
- `internal/gateslot/template.yaml`
- `internal/gateslot/config_test.go`

**Edit:**
- `internal/configreg#Modules`
- `internal/configreg/configreg_test.go`
- `cmd/lyx/configstrictness_test.go`
- `cmd/lyx/testdata/test-budget.yaml`
- `internal/gateslot/doc.go`

**Uses:**
- `internal/configengine#Load`
- `internal/landingshed#ConfigTemplate`

**Intent:**
Add the hub-wide `gate` config module (D5).
`template.yaml` holds `slots: 2`, `go_parallel: 4` and `cli_wait_sec: 300`, each with its trailing comment in the house style of `internal/landingshed/template.yaml`: `slots` bounds how many gate builds and tests the hub runs at once, `go_parallel` is the `-p` cap every slotted run gets, and `cli_wait_sec` is how long `lyx gate test` waits for a slot before its slot-busy exit.
`ConfigTemplate` embeds it the way `landingshed.ConfigTemplate` does.
`Config` mirrors the three keys; `LoadConfig(baseDir)` loads `gate` strictly through `configengine.Load` (`PATTERN-config-strictness`), naming `lyx fabric reconcile` as the way forward when the file is absent, and fails the load naming the key when any value is below 1.
`TemplateConfig` decodes the embedded template alone, for a run outside every hub, which applies the template's `-p` cap and no slot.
`Config.Limits` converts to the `Limits` the pool reads, `cli_wait_sec` as a duration.
`configreg.Modules` registers `gate` in alphabetical order with `HubWide: true` (`PATTERN-hub-wide-config`), so it is read and written only at `<BoardDir>/_lyx/config/gate.yaml` and the existing hub-wide reconcile materializes it from the template.
`configreg_test.go`'s hub-wide flag pin gains `gate`.
`cmd/lyx/configstrictness_test.go`'s strict pinned set gains `internal/gateslot`.
`config_test.go` is tier 1 and table-driven: the template loads to its defaults, and `slots: 0`, `go_parallel: 0` and `cli_wait_sec: 0` each fail naming that key; the gate package's budget row rises for it.
`doc.go` gains the config paragraph.

**ImpactSummary:** Registers one new hub-wide module; the registry fingerprint changes, so the next start after a binary change reconciles the hub and writes `gate.yaml` there, and no existing module changes.

# Card 2 — creel-config

**Create:**
- `plan:internal/creelconfig#ConfigTemplate` -> `func ConfigTemplate() string`
- `plan:internal/creelconfig#Config` -> `type Config struct`
- `plan:internal/creelconfig#ConfigSpec` -> `func ConfigSpec() configengine.Spec`
- `plan:internal/creelconfig#LoadConfig` -> `func LoadConfig(dirs configengine.Dirs) (Config, error)`
- `plan:internal/creelconfig_test#TestLoadConfig` -> `func TestLoadConfig(t *testing.T)`
- `internal/creelconfig/template.yaml`
- `internal/creelconfig/doc.go`
- `internal/creelconfig/config_test.go`
- `internal/creelconfig/testmain_test.go`

**Edit:**
- `internal/configreg#Modules`
- `internal/configreg#TestModules_SeedOnlyAndHubWideFlags`
- `internal/configreg#TestModules_LocalLayerRefusals`
- `cmd/lym#configStrictnessDegradingSet`
- `pattern/PATTERN-config-layers.md`
- `docs/overview.md`

**Uses:**
- `internal/configreg/doc.go`

**Intent:** Add the hub-wide `creel` config module with its one key `trusted_hubs` (D5), modelled on `internal/reportconfig`.

The template holds `trusted_hubs: []` with a comment saying it lists the shortnames of the hubs whose orch this hub obeys.
`Config` carries `TrustedHubs []string`.
`ConfigSpec` is hub-wide and classifies the key `configengine.CategoryGuard`, since it grants authority, so it lives in the hub's base file on the board and the local layer refuses it.
The module has no machine key, so no machine layer is stacked.
`LoadConfig` is a degrading caller, through `configengine.LoadOrTemplate`, so a hub with no `creel.yaml` trusts no hub.
It validates no shortname: a malformed entry matches no sender.
`PATTERN-config-layers`' degrading-caller list and `configStrictnessDegradingSet` gain `creelconfig`.
No reinterpretation is needed, since the module is new.

`Modules` registers `{Name: "creel", ...}` in name order, `HubWide: true`, with `CheckLoad` over `LoadConfig`, and the hub-wide chain of `TestModules_SeedOnlyAndHubWideFlags`, with its comment, adds `creel`.
`TestModules_LocalLayerRefusals` gains the row `{"creel", "trusted_hubs:\n  - ly\n", "trusted_hubs", false}`, which is D5's refused local file.

`internal/creelconfig/config_test.go` is tier0 and covers `TestLoadConfig` over a bare board: the template's empty list, a hub value, and malformed YAML returned as an error.
`internal/creelconfig/testmain_test.go` is untagged and calls `tmuxkit.Main`.

`docs/overview.md` gets a package-tree line for `internal/creelconfig/` and names it in the creel bullet.

**ImpactSummary:** One new hub-wide module, picked up by every `configreg.Names()` walk with no edit; `configreg` gains one import and the strictness scan one pinned package, and no reader uses the key until card 7.

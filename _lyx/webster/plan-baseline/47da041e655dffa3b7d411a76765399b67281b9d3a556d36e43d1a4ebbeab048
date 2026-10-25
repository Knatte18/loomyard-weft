# Card 1 — drift-config

**Create:**
- `internal/driftengine/doc.go`
- `internal/driftengine/template.yaml`
- `internal/driftengine/testmain_test.go`
- `plan:internal/driftengine#Config` -> `type Config struct`
- `plan:internal/driftengine#CommandStep` -> `type CommandStep struct`
- `plan:internal/driftengine#ConfigTemplate` -> `func ConfigTemplate() string`
- `plan:internal/driftengine#ConfigOpenMaps` -> `func ConfigOpenMaps() []string`
- `plan:internal/driftengine#ConfigSpec` -> `func ConfigSpec() configengine.Spec`
- `plan:internal/driftengine#LoadConfig` -> `func LoadConfig(dirs configengine.Dirs) (Config, error)`
- `plan:internal/driftengine#Config.Step` -> `func (c Config) Step(name string) (CommandStep, error)`
- `plan:internal/driftengine#Paths` -> `type Paths struct`
- `plan:internal/driftengine#NewPaths` -> `func NewPaths(anchorRoot string) Paths`
- `plan:internal/driftengine#DirRel` -> `func DirRel() string`
- `plan:internal/driftengine#TestLoadConfig` -> `func TestLoadConfig(t *testing.T)`

**Edit:**
- `internal/configreg#Modules`
- `cmd/lyx#configStrictnessStrictSet`

**Extends:**
- `docs/overview.md`

**Uses:**
- `internal/gateslot#ConfigSpec`
- `internal/darnengine#LoadConfig`

**Intent:**
Create `internal/driftengine`, the module that owns regression around a task (D1), starting with its hub-wide `drift` config module and its paths.

`template.yaml` declares, with one comment per key as `darnengine`'s template does:
`enabled: false`, `capture: {}`, `compare: {}`, `baseline_model: sonnet[medium]`, `verify_model: opus[high]`, `verify_max_runs: 8`.
`capture` and `compare` are each written whole as `{command, dir, env, timeout_min}`; the comment names `{baseline_dir}` (required) and `{skip_file}` (optional) and the `lyx config drift --set 'capture={...}'` form.

`ConfigSpec` names the module `drift`, `HubWide: true`, `OpenMaps` `capture` and `compare` (as `gateslot` does for `build`), categories `capture` and `compare` `CategoryExecutable` (template and board default layer only), every other key `CategoryNone`.
`LoadConfig` loads strictly through `configengine.Load`, mapping "not initialized" to the `lyx fabric reconcile` way forward as `darnengine.LoadConfig` does, and decodes each step like `gateslot`'s `decodeBuild`: an all-empty mapping is no step (zero `CommandStep`), a declared step needs `command` and `timeout_min` at least 1, `dir` defaults to `.`.
It refuses, naming the key and `lyx config drift` as the way forward:
a declared `capture` or `compare` whose command lacks `{baseline_dir}`; `timeout_min` below 1; `verify_max_runs` below 1; a `baseline_model` or `verify_model` that `modelspec.Parse` rejects.
`Config.Step(name)` returns the declared `capture` or `compare` step, and for an undeclared one refuses with `drift: <name> is not declared; way forward: declare it with lyx config drift --set '<name>={command: ..., timeout_min: N}'`; every Drift row and verb that runs a command calls it (D1).

`Paths` is the one declarer of every drift path, joined onto the told anchor root through `lyxdirs.LyxDirName` and `lyxdirs.DotLyxDirName` and a `driftDirName = "drift"` segment, as `websterengine.Dir`/`ScratchDir` do (PATTERN-durable-vs-ephemeral-state):
durable `_lyx/drift/` holds `Scope` (`scope.json`), `BaselineReport` (`baseline.md`), `VerifyReport` (`verify-report.md`) and `LogsDir` (`logs/`);
ephemeral `.lyx/drift/` holds `Capture` (`capture.json`), `CaptureLock` (`capture.lock`), `CompareMarker` (`compare.lock`), `BaselineDir` (`baseline/`), `SideDir` (`side/`), `SkipFile` (`skip.txt`) and `Runs` (`runs.json`).
`DirRel` returns `_lyx/drift`, forward-slashed, for a commit pathspec.

`doc.go` opens the package doc: the module's purpose, that regression is no test tier, the config keys and placeholders, and the two directories; later cards extend it.
`testmain_test.go` runs `gitkit.HermeticGitEnv()` and `tmuxkit.Main`, as `internal/hibernatecli`'s does, since later cards spawn git.

`configreg.Modules` gains `{Name: "drift", Template: driftengine.ConfigTemplate, OpenMaps: driftengine.ConfigOpenMaps(), HubWide: true, CheckLoad: checkLoad(driftengine.LoadConfig), Spec: driftengine.ConfigSpec}` in alphabetical place, so `drift.enabled` becomes a known setting for an entry gate (`configreg.CheckSettingKey`).
`cmd/lyx`'s `configStrictnessStrictSet` gains `internal/driftengine`, since the module runs only inside a hub.
`docs/overview.md`'s module tree gains the `internal/driftengine/` line.

`TestLoadConfig` (tier0) is table-driven over a hub file written into a `configengine.Dirs` temp board: the template alone loads with the defaults above and both steps undeclared; a declared capture loads with `dir` defaulted; each refusal above is a row asserting the error names the key; a local-layer file setting `capture` or `compare` is refused, since both are executable keys; dirs with no hub file refuse naming `lyx fabric reconcile`; `Step` on an undeclared step refuses naming `lyx config drift`, and on a declared one returns it; `DirRel` is `_lyx/drift` and `NewPaths`' durable files lie under it, so the rows' commit pathspec covers the scope record, the reports and the logs.

**ImpactSummary:** One new registry entry and one pinned-set entry; no existing module's load changes.

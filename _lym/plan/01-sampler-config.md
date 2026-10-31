# Card 1 — sampler-config

**Create:**
- `internal/samplerengine/doc.go`
- `internal/samplerengine/template.yaml`
- `internal/samplerengine/testmain_test.go`
- `internal/samplerengine/config_test.go`
- `plan:internal/samplerengine#Config` -> `type Config struct`
- `plan:internal/samplerengine#ConfigTemplate` -> `func ConfigTemplate() string`
- `plan:internal/samplerengine#ConfigSpec` -> `func ConfigSpec() configengine.Spec`
- `plan:internal/samplerengine#LoadConfig` -> `func LoadConfig(dirs configengine.Dirs, module string) (Config, error)`
- `plan:internal/samplerengine#TestLoadConfig` -> `func TestLoadConfig(t *testing.T)`

**Edit:**
- `internal/configreg#Modules`
- `internal/configreg#TestModules_SeedOnlyAndHubWideFlags`
- `internal/configreg#TestModules_EveryTemplateKeyClassified`

**Extends:**
- `docs/overview.md`

**Uses:**
- `internal/configreg/doc.go`
- `internal/darnengine#LoadConfig`
- `internal/darnengine#ConfigSpec`

**Intent:**
D3: a new hub-wide config module `sampler`, owned by the new package `internal/samplerengine`, loaded only through `configengine` (`PATTERN-config-layers`).
`template.yaml` holds two keys with one-line comments: `agent: opus[high]`, the model-spec of the sampler agent's interactive session, and `code_review: false`, whether the Code-Review perch runs on a sampler run.
`Config` mirrors them as `Agent string` (`yaml:"agent"`) and `CodeReview bool` (`yaml:"code_review"`).
`ConfigTemplate` returns the embedded template; `ConfigSpec` returns a `configengine.Spec` named `sampler`, `HubWide: true`, with both keys `configengine.CategoryNone`, so a task's `run_settings` or a profile block may set `sampler.code_review` in the local layer, as darn's `verify_attempts` is.
`LoadConfig` loads strictly through `configengine.Load` like `darnengine.LoadConfig`, maps "not initialized" to the `lym fabric reconcile` way forward as darn does, and refuses an `agent` value `modelspec.Parse` rejects, wrapping the error as `sampler config key "agent": …`.
The template default reaches every hub through the sparse hub base (`PATTERN-sparse-hub-base`); a hub with no `sampler.yaml` loads the template's values.
`doc.go` states the package: the `sampler` recipe's hub-wide config, its two keys, strict load, and that the package is told every path it reads and imports no resolver; later cards add the record and the session spec to it.
`testmain_test.go` is untagged and runs the package's tests under `tmuxkit.Main` (`PATTERN-test-isolation`).
`config_test.go` carries `//go:build tier0` and holds `TestLoadConfig`, a table over a hub board dir: no file loads `agent` `opus[high]` and `code_review` false (a hub with no file of its own); a hub file setting `code_review: true` loads true; a local layer setting `code_review: true` over the hub loads true; an `agent: "opus[[bad"` hub file is refused naming the `agent` key.
`configreg.Modules` registers `{Name: "sampler", Template: samplerengine.ConfigTemplate, HubWide: true, CheckLoad: checkLoadNamed(samplerengine.LoadConfig, "sampler"), Spec: samplerengine.ConfigSpec}` in sort order.
`TestModules_SeedOnlyAndHubWideFlags` adds `sampler` to the hub-wide names, and `TestModules_EveryTemplateKeyClassified` adds `"sampler": {"agent", "code_review"}` to `wantLocalKeys`; `TestRegistration_MatchesDeclarers` passes unchanged once the module is registered.
`docs/overview.md` gains a tree line for `internal/samplerengine/`, the sampler recipe's hub-wide `sampler.yaml` config.

**ImpactSummary:** one new module added to configreg's table and two of its table tests; no existing module's behaviour changes.

**Commit:** 1: samplerengine: a hub-wide sampler.yaml with agent and code_review

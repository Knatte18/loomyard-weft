# Card 3 — hardener-config

**Create:**
- `internal/hardenshed/doc.go`
- `internal/hardenshed/template.yaml`
- `plan:internal/hardenshed#Config` -> `type Config struct`
- `plan:internal/hardenshed#ConfigTemplate` -> `func ConfigTemplate() string`
- `plan:internal/hardenshed#ConfigOpenMaps` -> `func ConfigOpenMaps() []string`
- `plan:internal/hardenshed#ConfigSpec` -> `func ConfigSpec() configengine.Spec`
- `plan:internal/hardenshed#LoadConfig` -> `func LoadConfig(dirs configengine.Dirs, module string) (Config, error)`
- `plan:internal/hardenshed#CheckValues` -> `func CheckValues(document []byte) error`
- `plan:internal/hardenshed#TestLoadConfig` -> `func TestLoadConfig(t *testing.T)`
- `plan:internal/darnengine#NormalizeTrees` -> `func NormalizeTrees(trees map[string]string) (map[string]string, error)`

**Edit:**
- `internal/configreg#Modules`
- `internal/configreg#TestModules_SeedOnlyAndHubWideFlags`
- `internal/configreg#TestModules_EveryTemplateKeyClassified`
- `internal/configreg#TestPrepareRunSettings`

**Uses:**
- `internal/darnengine#normalizeTreeScope`
- `internal/darnengine/doc.go`
- `internal/configreg/doc.go`
- `internal/reviewbudget#Route`

**Intent:**
Add the `hardener.yaml` config unit (D9), loaded only through `configengine`, with its defaults in code.

`internal/hardenshed` (new) is the Hardener engine package (D11).
Its `doc.go` opens the package doc: the package holds the Hardener campaign's engine, and this card writes the paragraph on its config unit; card 20 adds crucible's method.
The package imports neither `internal/fabriccli` nor `internal/configreg`, so `configreg` can register its unit without a cycle; the CLI layer tells it what needs either.

`Config` mirrors the keys with these defaults, one line each in `template.yaml`:
`packages: []`, `llm_driving: false`, `max_bounces: 7`, `hard_max_bounces: 10`, `reviewer: [opus, fable, sonnet]`, `fixer: [opus, fable, sonnet]`, `judge: ""` and `seed: ""` (empty takes loom.yaml's `judge` and `seed`), `reviewer_fan: ""`, `fixer_fan: ""`, `auto_accept: false`, `concurrency: 3`, `live_attempts: 2`, `verify_attempts: 3`, `tree_scope: {}`.
`ConfigSpec` is `HubWide`; it classifies `auto_accept`, `reviewer_fan`, `fixer_fan` and `tree_scope` `CategoryGuard`, every other key `CategoryNone`.
Its `HubOnlyWayForward` names, for each Guard key, the write `lym config hardener --set <key>=<value>` from the prime, so a task's `run_settings` setting one is refused at seed with that way forward.
`ConfigOpenMaps` returns `tree_scope`.

`CheckValues` is the unit's value check over one document: it refuses a `concurrency`, `live_attempts`, `verify_attempts` or `hard_max_bounces` below 1, a negative `max_bounces`, and a `reviewer`, `fixer`, `judge` or `seed` entry whose model-spec does not parse, each naming the key.
`configreg` runs it on a local document at seed (`checkLocalDocument`), so a task whose `run_settings` hold a bad value is refused at seed, naming the task's `run_settings` as the place to correct.
`LoadConfig` loads strictly, runs `CheckValues` over the resolved document, and normalizes `tree_scope` through `darnengine.NormalizeTrees`; a refusal names the write for the layer the value comes from, through `reviewbudget.Route` over `ConfigSpec`, then `lym loom resume`.
`NormalizeTrees` is a thin exported wrapper over `normalizeTreeScope`, so both units take one tree-key rule; its errors wrap `darnengine.ErrTreeScopeKey`.

`configreg.Modules` gains the `hardener` row in its alphabetical place, `HubWide: true`, with `CheckLoad` over `LoadConfig` and `CheckValues` as its value check.
`TestModules_SeedOnlyAndHubWideFlags` and `TestModules_EveryTemplateKeyClassified` gain the module's rows: its local keys are the CategoryNone keys D9 lists as `run_settings` keys.
`TestPrepareRunSettings` gains rows: `run_settings` setting `hardener.packages` and `hardener.llm_driving` pass; one setting `hardener.auto_accept` is refused naming the hub file write; one setting `hardener.concurrency: 0` is refused naming the key.
`TestLoadConfig` is tier0: the template loads with every default above; a hub file sets every key; each bound refusal row names its key and its write; a malformed tree key wraps `ErrTreeScopeKey`.

**ImpactSummary:** One new config module registered in configreg and one exported wrapper in darnengine; no existing module's keys or loads change.

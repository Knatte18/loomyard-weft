# Card 7 — shed-config-module

**Create:**
- `internal/sheddriver/template.yaml`
- `plan:internal/sheddriver#ConfigTemplate` -> `func ConfigTemplate() string`
- `plan:internal/sheddriver#Config` -> `type Config struct`
- `plan:internal/sheddriver#ConfigSpec` -> `func ConfigSpec() configengine.Spec`
- `plan:internal/sheddriver#LoadConfig` -> `func LoadConfig(dirs configengine.Dirs) (Config, error)`
- `plan:internal/sheddriver#ValidateLocalValues` -> `func ValidateLocalValues(document []byte) error`
- `plan:internal/sheddriver#Override` -> `type Override struct`
- `plan:internal/sheddriver#CheckOverride` -> `func CheckOverride(module string, o Override) error`
- `plan:internal/sheddriver#CheckLocalOverride` -> `func CheckLocalOverride(module string, document []byte) error`
- `plan:internal/sheddriver#ResolveDriverKind` -> `func ResolveDriverKind(cfg Config, o Override) string`
- `plan:internal/sheddriver#ResolveTackler` -> `func ResolveTackler(cfg Config, o Override, reg modelspec.Registry) (TacklerSettings, error)`
- `plan:internal/sheddriver#TestLoadConfig` -> `func TestLoadConfig(t *testing.T)`
- `plan:internal/sheddriver#TestCheckOverride` -> `func TestCheckOverride(t *testing.T)`
- `plan:internal/sheddriver#TestResolveDriverKind` -> `func TestResolveDriverKind(t *testing.T)`
- `plan:internal/sheddriver#TestResolveTackler` -> `func TestResolveTackler(t *testing.T)`

**Edit:**
- `internal/configreg#Modules`
- `internal/configreg#TestPrepareRunSettings`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:**
Add the `shed` config module, `shed.yaml`, declared by `internal/sheddriver` and registered in `configreg` (D5): the shed layer's defaults for the Go driver and its tackler, which a module's own keys override from card 8.
Nothing reads it for a run yet; card 8 switches the seeding and arming sites over.

- `template.yaml` carries today's defaults with comments that say what each key does at the shed level: `driver_kind: llm`, `tackler: haiku`, `tackler_timeout_min: 10` (0 takes 10) and `tackler_max_spawns: 4` (0 turns the tackler off).
  `ConfigTemplate` embeds it; `Config` mirrors the four keys with plain `string` and `int` fields; all four live in `internal/sheddriver/config.go`.
- `ConfigSpec` names the module `shed`, not hub-wide, with every key `configengine.CategoryNone`, so the local layer and a task's `run_settings` may set them.
- `LoadConfig` loads through `configengine.LoadOrTemplate` (`PATTERN-config-strictness`), so a hub or pair with no `shed.yaml` reads the template, then validates the values through `CheckOverride("shed", ...)`; `ValidateLocalValues` is the module's `CheckValues`, checking the keys a local document sets through `CheckLocalOverride("shed", ...)`.
- `Override` is a module's override of the four keys: `DriverKind` and `Tackler` strings, where empty inherits; `TacklerTimeoutMin int`, where 0 inherits; and `TacklerMaxSpawns *int`, where nil inherits and a set 0 turns the tackler off.
  The pointer is the representation D5 asks the plan to pick: it tells an unset `tackler_max_spawns` (YAML null) from 0, as loom's per-perch budgets already do.
- `CheckOverride` refuses a `driver_kind` other than empty, `llm` or `go` (through `shedrun.ValidateDriver`, so the vocabulary stays `shedrun`'s), a non-empty `tackler` that `modelspec.Parse` rejects, a negative `tackler_timeout_min` and a negative `tackler_max_spawns`.
  Each refusal names `<module> config key "<key>"`, the bound, and how to set the key in that module's file or the task's `run_settings`, `null`/empty to inherit `shed.yaml`.
  `CheckLocalOverride` decodes a local document with pointer fields and applies the same checks to the keys it sets, marking the error `configengine.ErrInvalid`.
- Resolution is pure (`PATTERN-config-layers`: only `configengine` reads the files).
  `ResolveDriverKind` returns the override's driver kind, else `shed.yaml`'s, else `shedrun.DriverLLM`.
  `ResolveTackler` takes the model-spec from the override, else `shed.yaml`, else `haiku`; the timeout from a positive override, else a positive `shed.yaml` value, else 10 minutes; the spawn budget from a set override (0 included), else `shed.yaml`'s.
  It parses and resolves the spec through the registry and returns `TacklerSettings`, with an error naming the `tackler` key.
- `configreg.Modules` registers `{Name: "shed", Template: sheddriver.ConfigTemplate, CheckValues: sheddriver.ValidateLocalValues, Spec: sheddriver.ConfigSpec}` in sort order, after `reed`, so `TestRegistration_MatchesDeclarers`, `TestNames` and `TestModules_EveryTemplateKeyClassified` cover the registration.
  `PATTERN-sandbox-coverage` is untouched: the config module adds no cobra module, and the `shed` CLI module keeps its existing exclusion.

Tests:
- `TestLoadConfig`: no base layer anywhere reads the template defaults (llm, haiku, 10, 4); a base `shed.yaml` value and a local override are each read; one invalid value (`driver_kind: fast`) is refused naming `shed config key`, which proves the load runs `CheckOverride`, whose own test holds the per-value matrix.
- `TestCheckOverride`: rows for empty, `llm` and `go` driver kinds passing, each invalid value refused naming the module it was told, a nil spawn budget passing, and `CheckLocalOverride` refusing a local `tackler_max_spawns: -1` with `ErrInvalid` while a `null` passes.
- `TestResolveDriverKind`: override set wins; empty override takes `shed.yaml`'s; both empty takes `llm`.
- `TestResolveTackler`: empty override inherits every `shed.yaml` value; a set override wins on each key; an override `tackler_max_spawns: 0` gives 0 while `shed.yaml` says 4; an override timeout 0 takes `shed.yaml`'s, and a `shed.yaml` 0 takes 10 minutes; an unresolvable spec is refused naming `tackler`.
- `TestPrepareRunSettings` gains a row carrying `shed` `tackler_max_spawns` from `run_settings` into a `shed` document, and one refusing `shed` `tackler_max_spawns: -1` through its `CheckValues`.

`cmd/lyx/testdata/test-budget.yaml`'s `internal/sheddriver` row rises from 15 to 19.

**ImpactSummary:** one new config module registered in configreg with its run-settings test table extended; no reader switches in this card.

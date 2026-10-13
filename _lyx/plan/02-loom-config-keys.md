# Card 2 — loom-config-keys

**Edit:**
- `internal/loomengine#Config`
- `internal/loomengine#ConfigSpec`
- `internal/loomengine#LoadConfig`
- `internal/loomengine/template.yaml`
- `internal/loomengine#TestLoadConfig_Loads`
- `internal/loomengine#TestLoadConfig_Refuses`
- `internal/configreg#Module`
- `internal/configreg#Modules`
- `internal/configreg#PrepareRunSettings`
- `internal/configreg#TestPrepareRunSettings`
- `contracts/specs/refusal-spec.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Create:**
- `plan:internal/loomengine#TacklerSettings` -> `type TacklerSettings struct`
- `plan:internal/loomengine#ResolveTackler` -> `func ResolveTackler(cfg Config, reg modelspec.Registry) (TacklerSettings, error)`
- `plan:internal/loomengine#ResolveDriverKind` -> `func ResolveDriverKind(cfg Config) string`
- `plan:internal/loomengine#ValidateLocalValues` -> `func ValidateLocalValues(document []byte) error`
- `plan:internal/loomengine#TestResolveTackler` -> `func TestResolveTackler(t *testing.T)`

**Intent:**
D1's and D7's loom keys, every one open to the local layer so `run_settings` sets it per task.

`Config` gains four fields: `DriverKind string` (`driver_kind`), `Tackler string` (`tackler`), `TacklerTimeoutMin int` (`tackler_timeout_min`) and `TacklerMaxSpawns int` (`tackler_max_spawns`).
`template.yaml` gains the four keys beside the arbiter keys, each with its one-line comment: `driver_kind: llm` (`llm` or `go`, the driver a task's loom run is seeded with when nothing typed one; a seeded run keeps its recorded driver), `tackler: haiku`, `tackler_timeout_min: 10` (0 takes 10) and `tackler_max_spawns: 4` (0 turns the tackler off, so every stop it would take hands back).
`ConfigSpec` classifies all four `configengine.CategoryNone`, which `TestModules_EveryTemplateKeyClassified` and `TestConfigTemplate_ContainsEveryConfigYAMLTag` already check.

`LoadConfig` refuses, each naming the key and its way forward:
- a `driver_kind` other than empty, `llm` or `go` ("set driver_kind to llm or go in loom.yaml or the task's run_settings"), checked through `shedrun.ValidateDriver` so the vocabulary stays shedrun's;
- a `tackler` that is non-empty and does not parse as a model-spec;
- a negative `tackler_timeout_min` or `tackler_max_spawns`, in the same zero-is-a-meaning loop the arbiter knobs use.
The file-header comment of `internal/loomengine/config.go` names the four keys and their defaults beside the arbiter's.

`ResolveDriverKind` returns `shedrun.DriverLLM` for an empty `DriverKind` and the value otherwise; it is the one reader of the key, and only the seeding sites of cards 17 and 18 call it.
`ResolveTackler` mirrors `ResolveArbiter`: an empty `Tackler` takes `haiku`, a zero `TacklerTimeoutMin` takes 10 minutes, and `TacklerMaxSpawns` passes through; `TacklerSettings` carries `Model`, `Effort`, `Version`, `Timeout` and `MaxSpawns`.

`ValidateLocalValues` decodes a local-layer document into a partial `Config` and applies the `driver_kind` and negative-tackler-knob checks above to the keys the document sets, returning an error marked `configengine.ErrInvalid` (through `configengine.MarkInvalid`) that names the key.
`configreg.Module` gains `CheckValues func(document []byte) error`, called by `PrepareRunSettings` after `configengine.ValidateLocal` passes, its error wrapped in the same `*RunSettingsError` naming the module; the loom entry of `Modules` sets `CheckValues: loomengine.ValidateLocalValues`, and every other module leaves it nil.
So a task's `run_settings` holding `loom.driver_kind: fast` is refused by `PrepareRunSettings`, and Seed-Child's existing `ErrInvalidRunSettings` wrapping routes it to Stuck (D1).
Bound: `CheckValues` checks the four new keys only; every other key's value is still checked only at `LoadConfig`.

`refusal-spec.md` gains loom-section rows beside the `config: alignment_timeout_min` row: `config: driver_kind not llm or go`, `config: tackler not a model-spec` and `config: tackler knob negative`, each a correctness halt naming its way forward as the message does.
The `config: tackler not a model-spec` row is worded as the `config: arbiter not a model-spec` row is: its trigger is `loom.yaml`'s or a task's `run_settings` `tackler` being non-empty and no model-spec, or one the registry does not define, and its way forward is to set `tackler` to a model-spec the registry defines, or to empty to take `haiku`.

Tests:
- `TestLoadConfig_Loads` gains rows: the template loads with `DriverKind == "llm"`, `Tackler == "haiku"`, `TacklerTimeoutMin == 10` and `TacklerMaxSpawns == 4`; a local layer with `driver_kind: go` loads `go`.
- `TestLoadConfig_Refuses` gains rows for `driver_kind: fast`, `tackler: "not a spec["`, `tackler_timeout_min: -1` and `tackler_max_spawns: -1`, each asserting the key name in the error.
- `TestPrepareRunSettings` gains a `loom.driver_kind: fast` row expecting a refusal naming module `loom` and `wantInvalid`, a `loom.driver_kind: go` row expecting the document `driver_kind: go\n`, and a `loom.tackler_max_spawns: -1` row expecting a refusal naming module `loom`, the key `tackler_max_spawns` and `wantInvalid`.
- `TestResolveTackler`, beside `TestResolveArbiter` in `driver_test.go`: empty spec takes `haiku`, zero timeout takes 10 minutes, an explicit spec and timeout pass through, `MaxSpawns` passes through, and an unparseable spec is an error naming the tackler key.
`test-budget.yaml`'s `internal/loomengine` row rises by one.

**ImpactSummary:** four keys added to loom's config and template, one new optional hook on `configreg.Module` that only loom sets; existing hub `loom.yaml` files gain the keys on reconcile, and no existing key's validation changes.

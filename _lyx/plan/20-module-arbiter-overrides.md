# Card 20 — module-arbiter-overrides

**Edit:**
- `internal/loomengine#Config`
- `internal/loomengine#Config.ShedOverride`
- `internal/loomengine#LoadConfig`
- `internal/loomengine#TestLoadConfig_Loads`
- `internal/loomengine#TestLoadConfig_Refuses`
- `internal/loomengine/template.yaml`
- `internal/darnengine#Config`
- `internal/darnengine#Config.ShedOverride`
- `internal/darnengine#ConfigSpec`
- `internal/darnengine#TestLoadConfig`
- `internal/darnengine/template.yaml`
- `internal/battenshed#Config`
- `internal/battenshed#Config.ShedOverride`
- `internal/battenshed#ConfigSpec`
- `internal/battenshed#TestLoadConfig`
- `internal/battenshed/template.yaml`
- `internal/loomcli#arbiterDeps`
- `internal/loomcli#loomCLI.arbiterDepsFor`
- `internal/loomcli#arbiterSpec` -> `func arbiterSpec(prompt, verdictPath string, settings sheddriver.ArbiterSettings) shuttleengine.Spec`
- `internal/loomcli#arbiterCall.attachAndConclude`
- `internal/loomcli#arbiterCall.settleFresh`
- `internal/loomcli#arbiterFixture`
- `internal/loomcli#newArbiterFixture`
- `internal/loomcli#TestArbiterSpec`
- `internal/arbiterstop#HoldReading`
- `internal/arbiterstop#TestHoldReading`
- `internal/loomcli/doc.go`

**Delete:**
- `internal/loomengine#ResolveArbiter`
- `internal/loomengine#ArbiterSettings`
- `internal/loomengine#ArbiterTimeout`
- `internal/loomengine#arbiterDefaultModelSpec`
- `internal/loomengine#arbiterDefaultTimeout`
- `internal/loomengine#TestResolveArbiter`

**Uses:**
- `plan:internal/sheddriver#ArbiterSettings`
- `plan:internal/sheddriver#ResolveArbiter`
- `plan:internal/sheddriver#ArbiterLimits`

**Intent:**
Make `loom.yaml`, `darn.yaml` and `batten.yaml` each override `shed.yaml`'s arbiter keys (D8).
In all three templates the keys read `arbiter: ""`, `arbiter_timeout_min: 0` and `arbiter_max_spawns: ~`, commented as overrides of `shed.yaml`'s keys in the tackler overrides' words (an empty model-spec and a 0 timeout inherit, null inherits, a set 0 turns the arbiter off for that module); each `Config` carries `ArbiterMaxSpawns *int`, each `ConfigSpec` classifies the keys open to the local layer, and each `ShedOverride` passes the three keys, so `sheddriver.CheckOverride` judges them in every layer and `loomengine.LoadConfig` drops its own arbiter checks.
A hub layer that set a key in `loom.yaml` keeps its meaning as a loom override, an explicit 0 included, since the key name and value are unchanged.
`loomCLI.arbiterDepsFor` resolves the arbiter for the run's recipe as `tacklerFor` resolves the tackler: `sheddriver.ResolveArbiter(shed.yaml, loomengine.RecipeOverride(dirs, recipe), registry)`, so a darn run reads `darn.yaml`'s override and a loom or beam run `loom.yaml`'s; `arbiterDeps.Settings` and `arbiterSpec` take `sheddriver.ArbiterSettings`.
`arbiterstop.HoldReading` reads the child's recipe from its seed and takes `On`, `Timeout` and `SpawnsLeft` from `sheddriver.ArbiterLimits` over `shed.yaml` and that recipe's override, so batten's eligible-stop hold reads the same budget the child's driver spawns under.
`loomcli/doc.go`'s arbiter paragraph names `shed.yaml`'s keys and the recipe override.
Tests: `TestLoadConfig_Loads` and `TestLoadConfig_Refuses` rows read the override defaults and keep the negative and malformed refusals through `CheckOverride`'s texts; `darnengine`'s and `battenshed`'s `TestLoadConfig` gain an override row; `TestHoldReading` gains a row whose child is a darn run with `darn.yaml`'s `arbiter_max_spawns: 0`, read off; `TestArbiterSpec` and the arbiter fixture build `sheddriver.ArbiterSettings`.

**ImpactSummary:** Three module configs gain override keys and two readers move to shed's resolver; loomengine's arbiter resolver goes with its test, and its callers are all on this card.

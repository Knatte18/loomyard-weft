# Card 19 — shed-arbiter-config

**Edit:**
- `internal/sheddriver#Config`
- `internal/sheddriver#Override`
- `internal/sheddriver#ConfigSpec`
- `internal/sheddriver#LoadConfig`
- `internal/sheddriver#CheckOverride`
- `internal/sheddriver#CheckLocalOverride`
- `internal/sheddriver#TestCheckOverride`
- `internal/sheddriver#TestLoadConfig`
- `internal/sheddriver/template.yaml`
- `internal/sheddriver/doc.go`

**Create:**
- `internal/sheddriver#ArbiterSettings`
- `internal/sheddriver#ResolveArbiter`
- `internal/sheddriver#ArbiterLimits`
- `internal/sheddriver#TestResolveArbiter`

**Intent:**
Move the arbiter's keys into `shed.yaml` beside the tackler's, with today's loom defaults (D8).
`shed.yaml` gains `arbiter: opus[high]`, `arbiter_timeout_min: 30` and `arbiter_max_spawns: 3`, with comments saying a module's own key overrides each; `Config` and `ConfigSpec` carry them as per-worktree keys open to the local layer.
`Override` gains `Arbiter string`, `ArbiterTimeoutMin int` and `ArbiterMaxSpawns *int`, in the tackler overrides' shape: an empty model-spec and a 0 timeout inherit `shed.yaml`, a nil spawn budget inherits, and a set 0 turns the arbiter off for that module.
`CheckOverride` refuses a non-empty `arbiter` that is not a model-spec and a negative `arbiter_timeout_min` or `arbiter_max_spawns`, naming the module, the key and how to set it as for the tackler keys; `CheckLocalOverride` decodes the three keys too, and `LoadConfig` checks `shed.yaml`'s own values through it.
`ArbiterLimits` takes the timeout from the override, else `shed.yaml`, else 30 minutes, and the spawn budget from a set override, else `shed.yaml`'s; `ResolveArbiter` parses and resolves the model-spec through the registry, the override's else `shed.yaml`'s else `opus[high]`, and returns `ArbiterSettings` (a model choice through `shuttleengine.NewModelChoice`, the timeout and the spawn budget), with an error naming the `arbiter` key.
The package doc's config section names the arbiter keys and their override.
Tests: `TestResolveArbiter` (tier0, parallel) rows: shed defaults, a module model-spec and timeout override, a module `arbiter_max_spawns: 0` turning it off, a nil override inheriting shed's budget, an unparseable spec refused naming `arbiter`; `TestCheckOverride` gains the three refusals; `TestLoadConfig` reads the new template defaults.

**ImpactSummary:** shed.yaml and Override gain three keys; every Override literal keeps compiling with zero values, which inherit, so no module changes behaviour yet.

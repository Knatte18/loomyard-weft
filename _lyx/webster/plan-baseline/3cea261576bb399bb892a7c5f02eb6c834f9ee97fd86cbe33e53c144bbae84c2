# Card 2 — engine-config-key

**Edit:**
- `internal/shuttleengine#Config`
- `internal/shuttleengine#ConfigSpec`
- `internal/shuttleengine#LoadConfig`
- `internal/shuttleengine/template.yaml`
- `internal/shuttleengine_test#TestLoadConfig_TemplateDefaultsResolve`
- `cmd/lyx/testdata/test-budget.yaml`

**Create:**
- `internal/shuttleengine_test#TestLoadConfig_EngineKey`

**Intent:** Add the neutral `engine:` key to `shuttle.yaml`, the default engine for a spec that names none (D3).

`Config` gains `Engine string` with tag `yaml:"engine"`, its doc comment saying it names the engine of a spec with no engine and must be a registered engine name, checked where the registry builds the set.
`template.yaml` gains `engine: claude` above the `claude:` key, with a trailing comment: the engine for a spec that names none, which must be a registered engine name; an empty value is a config error.
`ConfigSpec` classifies `engine` as `configengine.CategoryNone`, so a local layer may set it and `shuttleengines.New` (card 5) is where an unregistered value from any layer is refused.
A `shuttle.yaml` without the key takes the template value through the normal layering, and hub reconcile adds it from the template like any new key; nothing else is needed for that.
`LoadConfig` refuses an empty `engine` after unmarshalling, beside its other value checks: `shuttle config: engine must name a registered engine, got an empty value`.
It does not check registration, since `shuttleengine` cannot see the registry (D2).

`TestLoadConfig_TemplateDefaultsResolve` asserts the template's `Engine` is `claude` beside its other template defaults.
`TestLoadConfig_EngineKey`, added to `internal/shuttleengine/config_test.go` beside the other key tests, is table-driven over a seeded `shuttle.yaml` built from `ConfigTemplate` with the `engine: claude` line replaced or removed: an override `engine: other` loads as `other`, a file with the line removed loads the template's `claude`, and `engine: ""` is refused with an error naming the `engine` key.
Raise `internal/shuttleengine` in `test-budget.yaml` from 173 to 174.

**ImpactSummary:** One new Config field and template key inside shuttleengine; every shuttle config load now refuses an empty engine, which no existing file holds.

# Card 4 — loom-arbiter-config

**Create:**
- `internal/loomengine#ArbiterSettings`
- `internal/loomengine#ResolveArbiter`
- `internal/loomengine#TestResolveArbiter`

**Edit:**
- `internal/loomengine#Config`
- `internal/loomengine#LoadConfig`
- `internal/loomengine/template.yaml`
- `internal/loomengine#TestLoadConfig_Loads`
- `internal/loomengine#TestLoadConfig_Refuses`
- `contracts/specs/refusal-spec.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:**
Add the arbiter's three `loom.yaml` keys (D7, D8) and their resolver.

`Config` gains `Arbiter string` (`arbiter`), `ArbiterTimeoutMin int` (`arbiter_timeout_min`) and `ArbiterMaxSpawns int` (`arbiter_max_spawns`).
`LoadConfig` refuses, each message naming the key: a non-empty `arbiter` that `modelspec.Parse` rejects, a negative `arbiter_timeout_min`, and a negative `arbiter_max_spawns`.
Zero and empty load cleanly, since a present `loom.yaml` lacking a key reads its Go zero value and each zero has a meaning:
an empty `arbiter` takes `opus[high]`, a zero `arbiter_timeout_min` takes 30 minutes, and a zero `arbiter_max_spawns` turns the arbiter off.

`template.yaml` gains `arbiter: opus[high]`, `arbiter_timeout_min: 30` and `arbiter_max_spawns: 3`, each with a comment stating its meaning and its zero meaning, and that `0` spawns turns the arbiter off so every eligible stop escalates to the run's parent as without it.

`ArbiterSettings`, in `driver.go` beside `DriverSettings`, carries `Model`, `Effort`, `Version`, `Timeout time.Duration` and `MaxSpawns int`.
`ResolveArbiter` substitutes `opus[high]` for an empty `cfg.Arbiter`, parses and resolves it through `reg` as `ResolveDriver` does, with an error naming the `arbiter` key, substitutes 30 minutes for a zero timeout, and passes `MaxSpawns` through.

Add rows to `TestLoadConfig_Loads` (keys absent read zero; keys present read their values) and to `TestLoadConfig_Refuses` (the three refusals, each asserting the key named).
`TestResolveArbiter` is table-driven: empty spec resolves `opus[high]`, a set spec resolves to it, zero timeout gives 30 minutes, a set timeout passes through, and an alias the registry lacks is an error naming the key.
`TestConfigTemplate_ContainsEveryConfigYAMLTag` covers the template lines unchanged.
Raise `internal/loomengine`'s row in the test budget by one.

Add loom-section rows to `contracts/specs/refusal-spec.md`: `config: arbiter not a model-spec` and `config: arbiter key negative`, each a correctness halt whose way forward names the key and its legal values.

**ImpactSummary:** three additive keys and validations in loom's config; no existing caller of `Config` or `LoadConfig` changes.

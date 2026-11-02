# Card 5 — module-fixer-keys

**Edit:**
- `internal/darnengine#ConfigSpec`
- `internal/loomengine#TapestryConfigSpec`
- `internal/loomengine#ConfigSpec`
- `internal/darnengine#Config`
- `internal/darnengine#LoadConfig`
- `internal/darnengine/template.yaml`
- `internal/darnengine#TestLoadConfig`
- `internal/loomengine#TapestryConfig`
- `internal/loomengine#LoadTapestryConfig`
- `internal/loomengine/tapestry.yaml`
- `internal/loomengine#TestLoadTapestryConfig`
- `internal/loomengine#Config`
- `internal/loomengine#LoadConfig`
- `internal/loomengine/template.yaml`
- `internal/loomengine#TestLoadConfig_Loads`
- `internal/loomengine#TestLoadConfig_Refuses`

**Uses:**
- `internal/darnengine/doc.go`
- `internal/loomengine#TestConfigTemplate_ContainsEveryConfigYAMLTag`

**Intent:**
Add one model-spec key per row caller (D3), each defaulting to `sonnet[medium]`, where an empty value turns that caller's fixer off.

- `darn.gate_fixer` in `internal/darnengine/template.yaml`, read into `darnengine.Config.GateFixer` (`yaml:"gate_fixer"`): the model-spec of the cold fixer of Darn-Write's verify gate.
- `tapestry.plan_gate_fixer` in `internal/loomengine/tapestry.yaml`, read into `TapestryConfig.PlanGateFixer`: the model-spec of the cold fixer of Plan-Write's plan gate.
- `loom.discussion_gate_fixer` in `internal/loomengine/template.yaml`, read into `loomengine.Config.DiscussionGateFixer`: the model-spec of the cold fixer of Discussion-Write's discussion gate.

Each module's spec classifies its new key as `CategoryNone`, as its sibling model-spec keys are, in `darnengine.ConfigSpec`, `loomengine.TapestryConfigSpec` and `loomengine.ConfigSpec`, since `configreg`'s template-coverage test refuses a template key no `Categories` entry covers.
Each template line carries a comment naming the gate the fixer serves and that `""` turns it off.
Each loader refuses a non-empty value `modelspec.Parse` rejects, in the shape its other model-spec keys refuse, naming the key and the way forward to set a model-spec with `lym config <module>` or `""` to turn the fixer off; an empty value loads as off.
The row gates read these keys through `Env.Setting` at build (card 7), so the typed fields serve the loaders' refusal and the writers' prompt choice (card 10).

Tests, tier0:
- `TestLoadConfig` (darn), `TestLoadTapestryConfig` and `TestLoadConfig_Loads` each gain a row asserting the template default `sonnet[medium]` and a row loading `""` as off.
- `TestLoadConfig` (darn), `TestLoadTapestryConfig` and `TestLoadConfig_Refuses` each gain a row refusing an unparsable value, naming the key.
- `TestConfigTemplate_ContainsEveryConfigYAMLTag` holds the two loom-side templates to their new tags unchanged.

**ImpactSummary:** three new optional template keys and their typed fields, each refused only when unparsable; no existing key or caller changes.

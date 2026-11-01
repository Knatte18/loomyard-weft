# Card 1 — shuttle-max-reruns-key

**Edit:**
- `internal/shuttleengine#Config`
- `internal/shuttleengine#ConfigSpec`
- `internal/shuttleengine#LoadConfig`
- `//internal/shuttleengine/template.yaml`
- `internal/shuttleengine_test#TestLoadConfig_TemplateDefaultsResolve`

**Extends:**
- `//internal/shuttleengine/doc.go`

**Create:**
- `plan:internal/shuttleengine_test#TestLoadConfig_KeepWarmMaxReruns` -> `func TestLoadConfig_KeepWarmMaxReruns(t *testing.T)`

**Uses:**
- `internal/loomcli#TestWire_EverySeamFilled`
- `internal/loomcli#TestWire_DarnRecipe`
- `internal/loomcli#TestWire_SamplerRecipe`
- `internal/loomcli#TestWire_BeamRecipe`
- `internal/battencli#TestWire_EverySeamFilled`
- `internal/testkit/envkit#TestFullEnv_FillsEverySeam`
- `internal/testkit/envkit#TestNilSeams_ReportsExactPaths`
- `internal/testkit/envkit#TestNilSeams_SkipsNilLegalPaths`

**Intent:** D5's break-even becomes a `shuttle.yaml` key.
`Config` gains the field `KeepWarmMaxReruns int` with yaml key `keep_warm_max_reruns`; `ConfigSpec` classifies it `configengine.CategoryNone`, so a local override may set it, per PATTERN-config-layers.
The template gains `keep_warm_max_reruns: 10` with a comment: the number of keep-warm re-runs one long gate wait is worth, since a re-run costs a cache read at about a tenth of the context's price against one cache write at about 1.25 times it; 0 turns the long wait off, and a negative value is a config error.
`LoadConfig` refuses a negative value with `shuttle config: keep_warm_max_reruns must not be negative, got <n>; way forward: set it to 0 or more with lym config shuttle`, as it refuses a negative `start_retries`.
The doc gains one sentence beside the `deny_python` sentence naming the key as the gate's break-even (D5).

Tests, both tier0 in `internal/shuttleengine`'s external test package:
`TestLoadConfig_TemplateDefaultsResolve` gains the assertion `cfg.KeepWarmMaxReruns == 10`.
The new `TestLoadConfig_KeepWarmMaxReruns` is a table over a local layer holding the key: `0` loads as 0, `3` loads as 3, and `-1` is refused with an error naming `keep_warm_max_reruns` and `lym config shuttle`.
The `//lym:seam` tests listed under Uses reflect over `Config`'s func-valued seams; an int field adds nothing they check, so they stay as they are.

**ImpactSummary:** one new optional shuttle.yaml key read by no caller yet; absent in older layers it resolves from the template, so every existing load keeps working.

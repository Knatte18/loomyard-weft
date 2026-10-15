# Card 4 — model-choice

**Create:**
- `plan:internal/shuttleengine#ModelChoice` -> `type ModelChoice struct`
- `plan:internal/shuttleengine#NewModelChoice` -> `func NewModelChoice(resolved modelspec.Resolved) ModelChoice`
- `plan:internal/modelspec#EngineNames` -> `func EngineNames() []string`
- `internal/shuttleengine/modelchoice_test.go`

**Edit:**
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:** Define the engine-neutral model choice `engine-model-choice` adopts (D11), and export `modelspec`'s closed engine-name set (D12).

In a new file `internal/shuttleengine/modelchoice.go`, `ModelChoice` has the string fields `Engine`, `Model`, `Effort` and `Version`, its doc comment saying it is one resolved choice of engine and model with the provider's effort and version, empty meaning the provider default.
`NewModelChoice` copies `resolved.Engine` and `resolved.Model` and reads `effort` and `version` from `resolved.Params`, an absent key giving an empty field.
`shuttleengine` may import `modelspec`, a leaf (`PATTERN-leaf-packages`); no caller adopts `ModelChoice` in this task.

In `internal/modelspec/modelspec.go`, `EngineNames` returns a sorted copy of `knownEngines`' keys, so a caller cannot change the closed set.
`modelspec` gains no import, so its leaf allowlist is untouched; the set stays `{claude}`.
Card 5's registry test pins `EngineNames` equal to the registry's names, which is its test.

`modelchoice_test.go` adds `TestNewModelChoice`, table-driven: a `Resolved` with both params maps all four fields; one with an empty `Params` map leaves `Effort` and `Version` empty; an unrelated param key is ignored.
Raise `internal/shuttleengine` in `test-budget.yaml` from 174 to 175.

**ImpactSummary:** Additive; the only edit raises one test-budget row.

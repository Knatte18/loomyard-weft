# Card 1 — board-label-vocabulary

**Edit:**
- `internal/boardengine/config.go`
- `internal/boardengine/template.yaml`
- `internal/boardengine/config_test.go`
- `internal/boardengine/template_test.go`

**Intent:**
The board config gains the two label lists D3 decides, and a `Vocabulary` value the store and the renderer read them through.

- `template.yaml` gains `types: [bug, enhancement]` and `labels: [undecided]`, each with a one-line comment: `types` are the type labels (a note carries exactly one, a task one or more), `labels` every other label.
  They are template defaults, not minimums (PATTERN-config-strictness): `boardengine` stays a strict `configengine.Load` caller, and a live `board.yaml` without the keys resolves to these defaults.
- `Config` gains `Types []string` (`yaml:"types"`) and `Labels []string` (`yaml:"labels"`).
- New exported type `Vocabulary` in `config.go`, holding `Types` and `Labels`, with `func (c Config) Vocabulary() Vocabulary` and two methods: `IsType(label string) bool` and `Known(label string) bool` (in either list).
  It is the one place that answers "is this a type label" and "is this label configured".
- `Outputs` gains `Types []string`, filled by `Config.Outputs`, so the renderer (card 3) can order the Notes sections by the configured type order without reading config itself.

Tests: `config_test.go` asserts `LoadConfig` resolves both lists from a config file that sets them, falls back to the template defaults when the keys are absent, and that `Outputs` carries `Types`;
`Vocabulary.IsType` and `Known` are tested for a type, a plain label and an unknown label, and an empty `Vocabulary` (a path-only `Config`) answers false to both.
`template_test.go` pins the two new keys and their defaults alongside the existing ones.

**ImpactSummary:** Additive config fields and one new value type in `boardengine`; no existing caller changes behaviour.

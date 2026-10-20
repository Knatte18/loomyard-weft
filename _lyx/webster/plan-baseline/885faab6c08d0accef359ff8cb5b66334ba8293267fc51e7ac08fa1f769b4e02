# Card 7 — darn reads lenses

**Edit:**
- `internal/darnengine#Config`
- `internal/darnengine#LoadConfig`
- `internal/darnengine/template.yaml`
- `internal/darnengine/doc.go`
- `internal/darnengine#templateConfig`
- `internal/darnengine#TestLoadConfig`

**Uses:**
- `plan:internal/lenses#FanList`
- `plan:internal/lenses#FanKey`
- `plan:internal/lenses#FanShapeError`
- `plan:internal/lenses#ValidateFanKeys`

**Intent:** darn's fan keys resolve through `lenses.yaml` (Scope: darn reads lenses and fans from `lenses`).
`Config`'s `ReviewerFan` and `FixerFan` become `lenses.FanList`, and their field comment names `lenses.yaml`; `LoadConfig` maps a `*lenses.FanShapeError` onto its key and validates both keys through `lenses.ValidateFanKeys`, so an entry naming an unresolvable fan is refused with `define the fan in lenses.yaml`; its doc comment names `lenses.yaml`.
`template.yaml`'s `darn_reviewer_fan` and `darn_fixer_fan` comments say "fan name from lenses.yaml"; `doc.go`'s two sentences naming `burler.yaml` for the fans name `lenses.yaml` and the `lenses` package's fan grammar.
Tests: `templateConfig` and `TestLoadConfig` build `lenses.FanList` values, and `TestLoadConfig`'s unknown-fan row expects `define the fan in lenses.yaml`.
`TestLoadConfig` gains a field for the board's `lenses.yaml` content, written beside the row's `darn.yaml` when set, and a row whose `darn_reviewer_fan` names a fan only that `lenses.yaml` defines, which loads with the fan key holding that name.

**ImpactSummary:** darnengine alone changes: two field types move package with identical YAML behaviour, and the refusal and comment texts name lenses.yaml; the one darn test row asserting the old text is updated.

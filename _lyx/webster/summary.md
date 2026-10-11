# plan-glyph-tuning: glyphs from the plan card to the Webster fork

Both batches reported `done`.

Batch 01 (cards 1–4, head `987cf43ea`) covered four cards:
- The plan template and plan-review rubric now name existing test members as glyphs.
- The code index builds a batch's pack of resolved member glyphs and unresolved refs.
- `BindHandles` returns a per-handle bound-or-failed outcome through a `planindex` type.
- The `plan:` handle ownership rule moved from `planglyph` into the exported `planparser.OwnedHandles`.

One deviation: `internal/planglyph/status_enforcement_test.go` was touched, because its allowlist of quarry-status readers needed a row for the new pack code.

Batch 05 (cards 5–8, head `faed67c7d`) covered four cards:
- Every terminal pass logs one line per owned `plan:` handle with its outcome and reason.
- `webster.yaml` gained the boolean `kickstart_pack`, template default true.
- The locate step and the pack section became stencils filled by `RenderForkPrompt`.
- `begin-batch` builds the pack into the fork prompt only and traces its state.

Deviations:
- `.gitattributes` gained `eol=lf` lines for the four new stencils.
- `internal/websterengine/render.go` holds the added pack-rendering helpers.

The `internal/websterengine` row in `cmd/lyx/testdata/test-budget.yaml` was raised to 202.

## Verify gate fixes

The gate failed once on `TestGofmt_ModuleFilesAreFormatted`: `internal/websterengine/template_test.go:604` had a misaligned `contains:` key.
The fixer realigned it in `fix:` commit `50c0630f7`, and reports `gofmt -l` clean and `lyx gate test --tags integration ./cmd/lyx ./internal/websterengine` passing.

## Gate steps

The forks ran each card's gate on the real packages, since the cards' gate steps named a non-directory `./internal/websterengine_test`.
The plan-level verify is left to the gate.

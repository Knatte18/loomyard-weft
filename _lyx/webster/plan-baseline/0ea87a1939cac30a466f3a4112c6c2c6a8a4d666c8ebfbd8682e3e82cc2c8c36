---
format: 5
approved: true
language: go
---

# Plan: plan-glyph-tuning — glyphs end to end, from the plan card to the Webster fork

Make glyphs work end to end, from the plan card to the Webster implementer fork.
Plan cards name the existing test members they change as glyphs.
Every terminal batch pass traces the outcome of each `plan:` handle bind.
`begin-batch` resolves each batch's member glyphs and renders the resolved spans into that batch's fork prompt, behind the `webster.yaml` switch `kickstart_pack`, so runs with and without the pack can be compared.

## Card Index

1 — plan-test-member-rule — the plan template and the plan-review rubric name existing test members as glyphs
2 — kickstart-pack-index — the code index builds a batch's pack of resolved member glyphs and unresolved refs
3 — bind-outcomes — BindHandles returns a per-handle bound-or-failed outcome through a planindex type
4 — owned-handles — the plan: handle ownership rule moves out of planglyph into the exported planparser.OwnedHandles
5 — bind-trace — every terminal pass logs one line per owned plan: handle with its outcome and reason
6 — kickstart-pack-switch — webster.yaml gains the boolean kickstart_pack, template default true
7 — pack-stencils-and-render — the locate step and the pack section become stencils RenderForkPrompt fills
8 — begin-batch-builds-pack — begin-batch builds the pack into the fork prompt only and traces its state

## Shared Decisions

- The pack reaches only the fork prompt file; the `begin-batch` envelope, its advisories and Master's context carry none of it (D6).
- `internal/websterengine` stays free of quarry: pack resolution and member-glyph selection live in `internal/planglyph` behind `planindex.Index`, and `planindex` keeps its standard-library-plus-`planparser` import set.
- The recovery prompt keeps today's quarry locate text and never takes a pack.
- With `kickstart_pack: false`, the fork prompt renders the same bytes as before this plan.

## verify:

```bash
go build ./...
go vet -tags integration ./internal/planparser/... ./internal/planindex/... ./internal/planglyph/... ./internal/websterengine/... ./contracts/stencils/... ./cmd/lyx/...
go vet -tags tmux ./internal/planparser/... ./internal/planindex/... ./internal/planglyph/... ./internal/websterengine/... ./contracts/stencils/... ./cmd/lyx/...
go vet -tags llm ./internal/planparser/... ./internal/planindex/... ./internal/planglyph/... ./internal/websterengine/... ./contracts/stencils/... ./cmd/lyx/...
go test ./internal/planparser/... ./internal/planindex/... ./internal/planglyph/... ./internal/websterengine/... ./contracts/stencils/... ./cmd/lyx/...
go test -tags integration ./internal/planparser/... ./internal/planindex/... ./internal/planglyph/... ./internal/websterengine/... ./contracts/stencils/... ./cmd/lyx/...
```

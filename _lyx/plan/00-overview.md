---
format: 5
approved: false
language: go
---

# Plan: Webster kick-start pack changes how a fork reads

The Webster kick-start pack becomes per card, and with the pack on, the fork starts each card with one parallel read of its card file and that card's listed spans.
The FRESH-READ rule then names those spans rather than whole files.
`var` and `const` entries shrink to the first line of their signature.
With `kickstart_pack` off, under a plan whose language opts out of glyphs, and in the recovery prompt, every rendered text stays byte-identical, so the with/without measurement compares like with like.
The pack stays a kick-start: the fork may read beyond it, and no source is inlined.

## Card Index

1 — pack-var-const-first-line — a var or const pack entry shows the first line of its signature only
2 — pack-per-card — the pack is resolved once per batch, grouped per card, and rendered one subsection per card under its pointer path
3 — pack-on-read-texts — step 1 and the FRESH-READ rule become markers with verbatim pack-off and new pack-on fragments

## verify:

```bash
go build ./...
go vet -tags integration ./internal/planindex ./internal/planglyph ./internal/websterengine ./contracts/stencils
go vet -tags tmux ./internal/planindex ./internal/planglyph ./internal/websterengine ./contracts/stencils
go vet -tags llm ./internal/planindex ./internal/planglyph ./internal/websterengine ./contracts/stencils
go test ./internal/planindex ./internal/planglyph ./internal/websterengine ./contracts/stencils
go test -tags integration ./internal/planindex ./internal/planglyph ./internal/websterengine ./contracts/stencils
```

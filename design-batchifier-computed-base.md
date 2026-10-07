# Batchifier: compute Merriam fixed base context, fit only its orientation

`batchifier-computed-base` · Note · enhancement, webster

Left from `batcher-v2`'s PR-Gate (2026-10-07).
`batcher-v2` sizes a batch's start context as `master_base + (k−1) × batch_growth`, with `master_base: 52000` a fitted constant in `internal/batcher/template.yaml`.
The operator wanted the base computed, since it is fixed by what Merriam loads.

Measured in `batcher-v2`'s Discussion over the three `cautious` runs: Merriam starts at about 20.6K and orients itself up to 50–54K before the first fork.
52K fits those three, but they are lyx tasks of similar plan size; a larger plan or another repo can land far from it.

## Proposal

- A function in `websterengine` computes Merriam's fixed part at partition time: CLAUDE.md, the Webster stencil with the inlined PATTERN overview, and the plan files Merriam reads; the system prompt and tools enter as a measured constant.
- `Batch` takes that value, beside `before`, and `startContext` becomes `computedBase + orientation + (k−1) × batch_growth`.
- `master_base` gives way to an `orientation` coefficient in `Weights`, `template.yaml` and the calibrate fit in `tools/tokencount`.

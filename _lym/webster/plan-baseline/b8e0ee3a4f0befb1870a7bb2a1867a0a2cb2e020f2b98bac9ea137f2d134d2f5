# Card 14 — pattern-docs-key-names

**Prosa:**
- `pattern/PATTERN-config-layers.md`
- `pattern/PATTERN-shed-driver.md`
- `pattern/PATTERN-landing.md`
- `docs/overview.md`
- `docs/shared-libs/configengine.md`
- `docs/reference/weaving-glossary.md`

**Intent:** `pattern/` and `docs/` outside `docs/research/` name every config key by its full dotted name (D3): `PATTERN-config-layers`' `loom.code_reviewer_fan` becomes `loom.code.reviewer_fan` and fabric's `branch_prefix` becomes `fabric.branch_prefix`; `PATTERN-landing`'s bare `pull_request` becomes `beam.pull_request`; `docs/overview.md`'s flat loom keys, `fabric.branch_prefix`, `gate.slots`, `gate.tier_parallel`, `codex.sandbox_mode` and `beam.pull_request` read in full.
`docs/shared-libs/configengine.md` gains the move table and `KeyName` beside the layer rules it already states.
The writer greps `pattern/` and `docs/`, `docs/research/` excluded, for each moved key's old name and each bare key; every further file holding one is part of this card.
`docs/research/` keeps the names of its runs.

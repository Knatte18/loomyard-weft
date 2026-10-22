---
round: 1
focus:
  - "Leftover pack references: grep the tree (code, stencils, docs, `docs/overview.md`, `PATTERN.md`, `pattern/`, `contracts/`, `.gitattributes`, `loom.yaml` and recipe files) for `kickstart`, `card-pack`, `CardPack`, `KickstartPack`, `pack-check`, `webster-fragment`; settle whether any live reference to a removed symbol, verb, key or stencil remains."
  - "Retired config keys in `internal/websterengine/config.go` and `configsync`: settle whether a layer still holding `kickstart_pack`, `kickstart_pack_per_card` or `kickstart_pack_file` loads and reconciles as the board entry and summary claim, and whether the retired-key mechanism matches how `kickstart_pack_file` was already handled."
  - "Inlined fork and recovery prompt text in `internal/websterengine/render.go` and the `webster-body-implementer` stencil: settle whether the inlined FRESH-READ rule, step 1, quarry locate step and sandbox rule match the former pack-off rendering, and whether the recovery prompt stays consistent with the fork prompt."
  - "`tools/forkaudit`: settle whether the non-pack sections (arm classification, trace cross-check, card-pack count, `--plan`) still work after the `--pack-check` and `--code` removal, and whether the board entry's 'keep working for runs without a pack' recommendation holds."
  - "Scope against the board entry: settle whether the diff removes anything beyond the pack (glyphs in the plan and plan gates, `lyx quarry`, the `docs/research/glyph-audit-*` reports must stay) and whether the entry's note on `enclose-checks` calls for a change the diff omits."
  - "Change description claims in `_lyx/landing/summary.md` (the removed-item lists, the marker sets, the test names): settle whether each is borne out by the diff."
---

Seed focus from the board entry and summary; the verify gate already decides build and test results.

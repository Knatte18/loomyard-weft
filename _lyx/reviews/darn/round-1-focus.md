---
round: 1
focus:
  - "Scope against the board entry: `buildKickstartPack` in `internal/websterengine/beginbatch.go` now returns nil for an empty pack and adds the `kickstart_pack=empty` log value. Settle whether the entry's 'empty pack' case licenses this, or whether the prompt already rendered as pack-off without it."
  - "Prompt layout in `contracts/stencils/webster/webster-body-implementer.md` and the two `webster-fragment-*` files: settle whether, with a pack, the pack section is the prompt's last section, and whether the batch-report section still says the report is the fork's last action without contradicting the new order."
  - "No-pack paths (pack off, language none, empty pack): settle whether the rendered prompt is unchanged from before, including the FRESH-READ rule text, which must not point at a pack section that is absent."
  - "Summary claims in `_lyx/landing/summary.md`: check each against the diff, especially 'without a pack the prompt is unchanged' and the claim that the instruction text moved from the fresh-read fragment into the pack-section fragment."
  - "Test economy: settle whether the edited tests in `contracts/stencils/claims_test.go`, `beginbatch_test.go` and `template_test.go` cover the empty-pack nil return and the pack-last ordering without duplicating each other."
  - "Docs: settle whether `internal/websterengine/doc.go` and the `WebsterFragmentPackSection` comment match the code's new placement and the log values."
---

Seed focus from the board entry (`webster-pack-last`) and the ten-file diff stat.

---
round: 1
focus:
  - "`internal/websterengine/beginbatch.go` `buildKickstartPack` against card 8: does it return no error, degrade (Warn, keep the pack from `deps.Plan` and `batch.Cards`) on a fresh-parse failure and on a `KickstartPack` error, return a nil pack for switch off and `language: none`, and emit exactly one `webster: kickstart pack` Info line with keys `batch`, `kickstart_pack`, `resolved`, `unresolved` in every case?"
  - "Pack isolation (D6): does the pack reach only the fork prompt file, with no new `BeginResult` field, no `Advisories` entry, and no pack in `RenderRecoveryPrompt`? Does `kickstart_pack: false` render byte-identical output to the pre-plan fork prompt?"
  - "Import boundaries: does `internal/websterengine` stay free of quarry, and does `planindex` keep its standard-library-plus-`planparser` imports after the `BindHandles` outcome type and the `planparser.OwnedHandles` move (cards 3 and 4)?"
  - "Bind trace (card 5): does every terminal pass log one line per owned `plan:` handle with outcome and reason, and does it use `planparser.OwnedHandles` rather than a second copy of the ownership rule left in `planglyph`?"
  - "Per-card mechanical checks: for each card's Rename, Delete and Edit groups, did the type-specific check run against every target in the group, such as callers of the old `planglyph` ownership helper after card 4?"
  - "Test economy: do `TestBeginBatch_KickstartPack` and the new `planindex`, `planglyph`, stencil and config tests duplicate existing coverage or test helpers the public surface already covers? Is any added behavior left without a test? Does `cmd/lyx/testdata/test-budget.yaml` reflect the new top-level tests?"
  - "Card 6 and 7: does `kickstart_pack` default to true in the template and the config, and do the locate step and pack section stencils keep today's locate text for unresolved member glyphs?"
---

Seed focus for round 1.
The entries name where to look; none caps severity or pre-states a verdict.

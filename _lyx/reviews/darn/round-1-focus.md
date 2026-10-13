---
round: 1
focus:
  - "Board entry says the pack comes with the one-turn instruction as the prompt's last section and the batch-report section stays above it; check the stencil order in `webster-body-implementer.md` and the rendered prompt end match, and that the batch-report text still names the report as the fork's last action."
  - "Without a pack (switch off, language none, empty pack) the board entry requires the prompt to read as today; check the render path leaves no stray heading, blank line or marker residue, and the recovery prompt never takes a pack."
  - "Empty-pack handling is new behavior beyond moving text (`kickstart_pack=empty`, no pack file); check it against the entry's scope and that the begin-batch write order (pack file before prompt) and nil-pack return of `RenderForkPrompt` are correct, including error paths."
  - "Claims in `_lyx/landing/summary.md` about unchanged bytes without a pack, the FRESH-READ rule pointing at the pack file, and the pack locate step wording: verify each against the diff."
  - "Docs and test economy: `internal/websterengine/doc.go` updated for the pack file, read line and `empty` state; new or changed tests do not duplicate existing coverage, and `cmd/lyx/testdata/test-budget.yaml` is edited if a package passes its row."
---

Seed focus for round 1, drawn from the board entry's Change section and the summary's claims.

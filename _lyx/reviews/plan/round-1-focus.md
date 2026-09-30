---
round: 1
focus:
  - "Card 10 (stencil-verdict-handoff) edits `contracts/stencils/landing/landing-template-describe.md`, but the decision record's Scope names only the integration fork and Master stencils. Settle whether D11 (summary.md as the channel to the PR) licenses the Describe stencil change, or whether it is scope the decision record does not grant."
  - "Card 11 (triage-docs) asserts that no module doc under `manifest/designs/` describes the integration stage, so none changes. The decision record's Scope requires any such doc to be updated. Settle the claim against the repo's `manifest/designs/` contents."
  - "Card 9 (stage-wiring) takes the D7 baseline as `batches[0]`'s recorded `StartSHA`. Settle whether that field is the HEAD captured before the first batch in execution order began, including after a resumed or re-begun batch 1, and whether an absent record or empty `StartSHA` routes to D8's baseline-unavailable path as the card says."
  - "Cross-card warning contract between card 5 (`triageOutcome.Warnings`), card 8 (`triageWarnings`), and card 9 (stage return). Settle whether D8's two warnings ('could not be localized' and 'baseline comparison unavailable') are each produced exactly once for both the nil-bisector case and the bisector-present-but-empty-baseline case, with the latter still localizing through bisect."
  - "Granularity of cards 3, 7, and 8. Settle whether each card is one independently reviewable and testable unit. Card 3 bundles the capture seam with the log-path helpers, card 7 bundles two summary appenders, and card 8 bundles three helpers."
  - "Acceptance-criteria coverage. For each criterion in the decision record, settle whether some card's named tests exercise it. Check in particular 'a Master done with no integration report still fails loud' after card 9 removes the sibling fail-loud error, and the baseline-restore criterion split between card 5 (run error) and card 9 (dirty tree)."
---

The plan appears to carry every Decision D1–D11.
The open questions are the edges: scope beyond the decision record (the Describe stencil), a factual claim about `manifest/designs/`, the provenance of the baseline SHA, and whether the split across cards 5, 8, and 9 composes into exactly the warnings, reasons, and records the decision record specifies.

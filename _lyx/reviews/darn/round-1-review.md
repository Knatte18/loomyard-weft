---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: internal/planglyph/deleteorder.go:161
    summary: Deleting cards numbered above a deleting card become later-card holders, so a same-batch higher-numbered card's region is newly reported by the post-batch and recover-batch callers
    fix: In LaterDeleteReferences keep the cards added from deleting (those not already in later) usable only as exemption regions, never as reported holders. For example, record the card numbers of later and have deleteOrderFindings/laterEditRegions report only holders whose card is in later, while unrewrittenLaterEditRegions still consults every region for the card.Number <= cardNumber exemption. Add a TestLaterDeleteReferences row where a batch holds two deleting cards, the second numbered higher and referencing the first's deleted member in its own Edit span, expecting no finding.
---

### [MEDIUM] Same-batch higher-numbered deleting cards are now reported as later holders

**Location:** internal/planglyph/deleteorder.go:161
**Issue:** The change adds every card of `deleting` that is not already in `later` to the Edit-region set, to get the exemption regions. `deleteOrderFindings` then reports from the same set through `laterEditRegions`, which matches any region whose card number is above the subject's.
Webster's `recordbatch.go:152` and `recoverbatch.go:499` call `LaterDeleteReferences` with `own` (a batch that already has a record) and `unbegunCards`, which skips recorded batches, so `own` is not in `later` there.
For a multi-card batch where card 5 deletes a member and card 7 of the same batch has an Edit span that still references it, card 7's region is now built and the reference is reported blocking, naming card 7 as an unbegun later card. Before the change card 7's region was never built, so nothing was reported.
The board entry asks only for skipping references that the deleting or an earlier card rewrites; it does not license reporting same-batch cards. The signature and doc comment also define the reported holders as cards in `later`.
I read the call paths and did not run a repro, so the exact outcome for a real batch is unconfirmed.

## Checked, no finding

- Off-by-one: `unrewrittenLaterEditRegions` exempts with `r.card.Number <= cardNumber`, which covers the deleting card and earlier ones, and a line held only by a later card is still reported.
- `editRegion.holds` keeps the old containment test exactly, so `laterEditRegions` and the caller-uncovered hand-off keep their meaning.
- A card in both `deleting` and `later` is added once (dedup by number), and `ValidateDispatch` passes the same slice for both.
- The unreadable-answer path for a deleting card's Edit target matches the summary's Risk section.
- Tests: no new top-level test function was added, so the `internal/planglyph` row in `cmd/lyx/testdata/test-budget.yaml` is not pushed; the three cases are pinned by the new `TestLaterDeleteReferences` rows and the changed `TestGlyphChain_CallerCoverage` row.
- The `doc.go` entry states the exemption and the `laterImportFindings` non-exemption accurately.

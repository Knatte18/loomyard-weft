---
round: 1
ledger:
  - key: card20-coverage-citation-unsupported
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: card17-orch-stencil-check-outcome-unstated
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
  - key: card21-watcher-table-underspecified
    rounds: [1]
    status: open
    class: design
    severity: LOW
---

Round 1 raised three findings, all open.
The gating one is the card 20 coverage citation: the deleted `Runner` delegation tests are not covered by the tests the card names.
Card 17 leaves the check of the remaining orch stencil sections without a stated outcome.
Card 21 does not say how the watcher table rows express their differing steps.

---
round: 1
ledger:
  - key: deleting-cards-reported-as-later-holders
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
---

Round 1 raised one finding (F1).
`LaterDeleteReferences` adds deleting cards to the Edit-region set to get exemption regions, but the same set feeds the reporting path, so a higher-numbered same-batch deleting card is reported as a later holder.
The fix separates exemption-only regions from reportable holders, which changes the function's contract, so it needs a fresh review.

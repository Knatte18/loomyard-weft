---
round: 1
ledger:
  - key: redundant-testdrivenotices-watched-blocked-row
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised F1: the `watched blocked stop sends nothing` row duplicates `watched awaiting without a notice sends nothing`.
The fix is a local row deletion.

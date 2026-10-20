---
round: 1
ledger:
  - key: summary-overclaims-reapsuccesswait-bound
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised F1: the summary says the constant bounds every expected-success wait, but three process-exit waits keep their own bounds.
The waits are sound; only the summary's claim needs narrowing.

---
round: 1
ledger:
  - key: strand-listing-under-store-mutex
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: notifier-wirings-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: removed-shuttle-send-named-in-doc
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised three low-weight findings (F1, F2, F3), mapped to the keys above in order.
None gates, so the loop converged in its first round.

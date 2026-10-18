---
round: 1
ledger:
  - key: reconcile-default-layer-rules-file-untested
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
---

Round 1 raised one LOW scope finding (F1): no test pins that `reconcile --layer default --apply` leaves an uncommitted hub-wide rules file alone.
The fix is a local assertion in an existing test.

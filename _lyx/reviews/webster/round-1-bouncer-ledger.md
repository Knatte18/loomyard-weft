---
round: 1
ledger:
  - key: stale-file-headers-attribute-deviations-to-fork
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised one finding (F1): two file headers in `internal/websterengine` still call the deviation list the fork's own, though the scope guard now fills it.
The fix text names the exact replacement wording, so it is mechanical.

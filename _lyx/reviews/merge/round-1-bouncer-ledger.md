---
round: 1
ledger:
  - key: recordless-paths-untested-multi-engine
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
---

Round 1 raised one finding (F1): no test pins that a strand with no run record is driven through the set's default engine on `Send`, `Interrupt` and `SessionIdle`.
It is a local test addition whose content the finding text fixes.

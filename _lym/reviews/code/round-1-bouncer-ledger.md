---
round: 1
ledger:
  - key: beam-orch-guard-unpinned-branches
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised F1, mapped to `beam-orch-guard-unpinned-branches`: the numbered orch role and a non-string `project` branch of the guard lack test rows.
The fix is two rows in the existing table, so it is mechanical.
It is LOW and does not gate.

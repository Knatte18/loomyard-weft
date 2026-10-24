---
round: 1
ledger:
  - key: removal-first-naming-card-crediting-unpinned
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
---

Round 1 raised one finding (F1): the `TestNewPlanGate` table has no row showing a `removes` entry is credited to the first pending card naming its ID.
The fix is a local test row whose content the finding text fixes.

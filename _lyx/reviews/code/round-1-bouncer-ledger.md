---
round: 1
ledger:
  - key: turn-end-newest-main-chain-row-missing
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: sidechain-exclusion-row-missing
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: usage-limited-logs-for-non-notifying-turn-ends
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised three LOW findings and nothing gating.
F1 and F2 ask for two more rows in `TestTurnEndUsageLimited`.
F3 asks to narrow or reword the `usageLimited` case in `wait.go`.

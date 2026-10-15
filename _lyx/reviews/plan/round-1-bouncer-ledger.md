---
round: 1
ledger:
  - key: judge-part-placement-pinned-one-side
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised F1 only: card 4's wants for the judge part pin its trailing edge at one host, not its leading edge.
The fix is a local move of one want into the shared wants and adds one, with no target or verify change.

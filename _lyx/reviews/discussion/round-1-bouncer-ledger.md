---
round: 1
ledger:
  - key: d1-no-status-condition-on-written-task
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: mechanical
  - key: d1-abandoned-sibling-unstated
    rounds: [1]
    status: open
    class: decision
    severity: LOW
    fix: mechanical
  - key: d3-shared-dependency-set-ambiguous
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised three findings against D1 and D3 of the decision record, none gating.
Each finding's text already fixes its correction, so all three are mechanical.

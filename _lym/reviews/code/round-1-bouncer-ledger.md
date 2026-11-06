---
round: 1
ledger:
  - key: code-side-archived-judgement-exemption-untested
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: archived-judgement-base-name-comparison-unpinned
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
---

Round 1 raised two LOW test-coverage findings on the judge snapshot's archived-judgement exemption (F1, F2).
Both name the exact test row to add, so both are mechanical.
Neither is gating.

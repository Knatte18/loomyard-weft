---
round: 1
ledger:
  - key: d9-budget-exit-misnamed
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: d8-at-entry-settle-unbounded
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
  - key: d7-at-entry-sentence-misdescribes-bound
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: d2-seed-precedence-no-criterion
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: d9-rationale-carries-rejected-alternative
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
---

Round 1 raised five findings, all open.
The gating one is the D9 budget exit: repeated refused judgments end in Shed's budget-exhausted block, not a parent escalation.
The rest bound D8's at-entry settle path, fix D7's at-entry sentence, add a seed-precedence criterion and move D9's rejected-alternative text to the support log.

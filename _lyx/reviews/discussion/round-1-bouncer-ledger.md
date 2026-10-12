---
round: 1
ledger:
  - key: replay-vs-harvest-auto-accept-outcome
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
  - key: soft-hard-cap-measure-undefined
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: step4-returning-key-test-underdefined
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: mechanical-mistag-bound-missing
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: budget-escalation-at-hard-cap-offers-continue
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
  - key: hard-cap-escalation-over-recorded-continue-unanswerable
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: orch-stencil-change-has-no-decision
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
  - key: rule-continue-cause-unstated
    rounds: [1]
    status: open
    class: decision
    severity: LOW
---

First round; no earlier ledger.
Eight findings map one to one to the keys above (F1 to F8 in order).
The cap-measure finding (F2) underlies several others (F3, F5, F6), so a fix there may shift them.

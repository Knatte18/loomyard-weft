---
round: 1
ledger:
  - key: pr-open-help-omits-new-refusals
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: judge-openable-order-and-behind-only-unpinned
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: judge-openable-failure-paths-uncovered
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 is the first round, so no earlier ledger exists.
F1 maps to the help-text key, F2 to the refusal-order key, F3 to the failure-path key.
None is gating, so the loop converges.

---
round: 1
ledger:
  - key: attach-if-live-gated-removes-nothing-untested
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: runner-shuttle-gate-forwarding-untested
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
---

Round 1 is the first round, so there is no earlier ledger.
F1 maps to the removes-nothing key and F2 to the gate-forwarding key.
Both are non-gating `scope` findings about missing test coverage.

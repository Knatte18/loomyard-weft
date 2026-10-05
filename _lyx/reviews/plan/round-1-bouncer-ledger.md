---
round: 1
ledger:
  - key: card1-misses-settings-matrix-test
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
  - key: card3-envelope-path-source-stated-once
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
  - key: overview-step-envelope-sentence-stale
    rounds: [1]
    status: open
    class: design
    severity: LOW
  - key: last-step-test-either-or
    rounds: [1]
    status: open
    class: decision
    severity: NIT
---

Round 1 is the first review, so there is no prior ledger.
All four findings are open and none is gating.
F1 and F2 are MEDIUM in non-gating classes, F3 is a LOW design finding, and F4 is a NIT.

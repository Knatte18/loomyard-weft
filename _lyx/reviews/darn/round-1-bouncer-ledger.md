---
round: 1
ledger:
  - key: archive-schedule-comment-lock-claim
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: archive-schedule-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: pushwithretry-empty-delays-row-missing
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised three findings.
The archive schedule comment and summary wrongly say the teardown holds no lock other writers wait on, while `Remove` pushes under the pair's exclusive teardown lock (F1).
The archive push's own delay schedule has no test (F2).
`PushWithRetry` has no empty-delays row (F3).

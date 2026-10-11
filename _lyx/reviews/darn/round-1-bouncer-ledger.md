---
round: 1
ledger:
  - key: shared-escalation-brief-changed-for-loom
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: busy-lock-rename-path-uncovered
    rounds: [1]
    status: open
    class: scope
    severity: LOW
  - key: wiredarn-ignores-isbouncer-result
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
---

Round 1 was the first round, with no prior ledger.
It raised three non-gating findings: the shared escalation brief's darn wording reaches loom runs (F1), the busy-lock warn path in `renameRetiredDarnRows` is untested (F2), and `wireDarn` discards the `isBouncer` result (F3).

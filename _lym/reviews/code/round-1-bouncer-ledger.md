---
round: 1
ledger:
  - key: wait-reminder-post-seam-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: cancel-marker-and-anchor-change-rows-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 (first round, no prior ledger).
F1 maps to `wait-reminder-post-seam-untested`: the Post seam that reaches the operator is checked only for non-nil.
F2 maps to `cancel-marker-and-anchor-change-rows-untested`: no test row for the cancel-marked run skip or the standing anchor change.
Both fixes are test additions the finding text already specifies.

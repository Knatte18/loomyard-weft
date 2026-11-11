---
round: 1
ledger:
  - key: adopt-check-failed-without-failedtier-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: reed-refusal-accepted-path-and-down-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised two LOW design findings, both missing test rows.
F1 maps to `adopt-check-failed-without-failedtier-untested` and F2 to `reed-refusal-accepted-path-and-down-untested`.
Neither is gating, so the loop converges.

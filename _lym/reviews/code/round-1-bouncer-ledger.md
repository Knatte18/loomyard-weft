---
round: 1
ledger:
  - key: dobbyruns-ended-pair-wiring-unpinned
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: binary-check-hold-and-nil-teardown-unpinned
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: retry-ceiling-and-wait-forgetting-unpinned
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised three LOW test-coverage findings.
F1 maps to `dobbyruns-ended-pair-wiring-unpinned`, F2 to `binary-check-hold-and-nil-teardown-unpinned`, F3 to `retry-ceiling-and-wait-forgetting-unpinned`.
There is no earlier ledger, and no key recurs.

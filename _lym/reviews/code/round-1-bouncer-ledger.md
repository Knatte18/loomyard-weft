---
round: 1
ledger:
  - key: uncomputed-mergeability-halt-advice
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: merge-state-read-failure-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: retry-timing-docs-understate-wait
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: beam-docs-miss-resume-only-exception
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: refusalother-contract-false
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: merge-danger-blast-radius-narrow
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised six findings, none carried from earlier rounds.
The two gating ones are F1, an uncomputed mergeability that outlasts the poll still halting with approval and cancel advice, and F2, an untested failed merge-state read inside the merge loop.
F3 to F6 are doc and summary consistency items at LOW.

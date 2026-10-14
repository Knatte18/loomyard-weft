---
round: 1
ledger:
  - key: open-map-list-shape-overwritten-by-set
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: trailing-text-after-bracket-dropped
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: summary-overstates-sparse-test-rows
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised three findings.
A mapping-declared open map holding a list takes a scalar at its own path because the shape check runs after the file's value is carried in (F1).
A bracketed value with trailing text is written as the list alone (F2).
The change description claims sparse-path test rows that do not exist (F3).
All three have fixes the review text already fixes.

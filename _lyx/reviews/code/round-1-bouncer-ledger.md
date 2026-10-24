---
round: 1
ledger:
  - key: anchored-dotdot-clamped
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: git-init-relative-dotdot-recorded
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: missing-rows-added-behaviours
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: doc-omits-git-init-bound
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 is the first round, so no earlier ledger exists.
The four findings are all LOW or NIT, and none gates.
Each has a local fix the review text already fixes.

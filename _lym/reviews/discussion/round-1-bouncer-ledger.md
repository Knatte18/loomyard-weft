---
round: 1
ledger:
  - key: design-prefix-refusal-too-narrow
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: rename-move-point-in-write
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: open-risk-names-settled-coordination
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: older-binary-rename-flag-without-file
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: merge-remove-reupsert-drops-flag
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: d2-bound-omits-hand-route
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: done-entry-way-forward-refused
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: goto-past-bounds-disagree
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: carry-over-overstated-board-copy
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: failed-commit-paths-uncovered
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: d9-arbiter-test-undefined
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 is the first round, so no earlier ledger exists to carry forward.
Two gating design findings are open at MEDIUM: the `design_prefix` refusal that leaves record files open to the render, and the rename move with no stated point in the write.
One MEDIUM `decision` finding, the Open risks line, goes to the run's parent.
Eight LOW findings remain, three of class `design` and five of class `consistency`.

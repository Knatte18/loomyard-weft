---
round: 1
ledger:
  - key: nested-block-rewrite-order
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: block-done-two-events
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: goto-bypass-block-gate-unbounded
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: recipe-name-awaits-operator
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: mechanical
  - key: skipped-route-reader-goto-admission
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: block-on-done-points-into-own-block
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: d8-bound-denies-budgets
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised seven findings, none recurring.
The gating pair is F1 (`nested-block-rewrite-order`) and F3 (`goto-bypass-block-gate-unbounded`).
F4 (`recipe-name-awaits-operator`) waits on the parent and operator.
F2 maps to `block-done-two-events`, F5 to `skipped-route-reader-goto-admission`, F6 to `block-on-done-points-into-own-block`, F7 to `d8-bound-denies-budgets`.

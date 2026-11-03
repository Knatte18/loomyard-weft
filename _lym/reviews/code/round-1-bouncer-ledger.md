---
round: 1
ledger:
  - key: whole-map-text-field-kind-unchecked
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: whole-map-branch-order-differs
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: parent-path-branch-untested
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: comment-preservation-untested
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: known-keys-compact-untested
    rounds: [1]
    status: open
    class: scope
    severity: NIT
    fix: mechanical
---

Round 1 raised five low-weight findings (F1 to F5, in the order above) and no gating one.
They are one missing text-field check, one branch-order inconsistency and three test gaps.

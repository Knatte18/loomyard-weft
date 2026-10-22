---
round: 1
ledger:
  - key: added-surface-beyond-entry
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: status-draining-true-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: help-text-upto-default
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised three LOW findings.
F1 (added surface) is deferred to the parent, not reverted.
F2 asks for a test of `draining: true` through `lyx gate status`.
F3 asks the `test` help text to say `--upto` alone replaces the tier0 default.

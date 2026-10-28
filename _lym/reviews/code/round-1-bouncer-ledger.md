---
round: 1
ledger:
  - key: output-above-header-ignores-package
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: retry-wiring-uncovered
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: failures-first-entry-unbounded
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: acceptance-criteria-partly-asserted
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: retry-delay-cancel-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised five findings, all `design`.
F1 (`output-above-header-ignores-package`) and F2 (`retry-wiring-uncovered`) are MEDIUM and gate.
F3 to F5 are LOW.
Each fix is a local correction whose result the finding text fixes.

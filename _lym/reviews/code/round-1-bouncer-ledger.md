---
round: 1
ledger:
  - key: gateslot-doc-tier-table-scope
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: coverage-selector-setup-untested
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: selected-run-execution-untested
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: computescope-coverage-branch-untested
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: selection-recorded-unasserted
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: gatecli-doc-sample-scope
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: codeworktreechanges-not-in-summary
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: guard-scan-error-mislabelled
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: targetsfor-boundary-rows
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: narrow-coverage-rows-incomplete
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
---

Round 1 raised ten findings, none BLOCKING.
Five are MEDIUM: one stale doc table (F1) and four test coverage gaps (F2 to F5).
Five are LOW: a stale doc sample (F6), an undisclosed fabric method (F7), a mislabelled guard-scan miss (F8, the only design-class finding) and two test row gaps (F9, F10).
No gating finding stands, so the loop converges.

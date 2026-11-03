---
round: 1
ledger:
  - key: config-reconcile-prune-flag-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: automatic-callers-prune-flag-unpinned
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: prunesparse-skips-prune-pass
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: preserved-report-promise-overstated
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: pattern-wording-covers-standalone-base
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

First round, no earlier ledger.
F1 maps to `config-reconcile-prune-flag-untested` and is the only gating entry.
F2 maps to `automatic-callers-prune-flag-unpinned`, F3 to `prunesparse-skips-prune-pass`, F4 to `preserved-report-promise-overstated`, F5 to `pattern-wording-covers-standalone-base`.
Every fix is a local correction whose result the finding text states.

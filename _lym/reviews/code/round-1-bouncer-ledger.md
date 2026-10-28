---
round: 1
ledger:
  - key: latest-darn-outcome-goto-skipped-entries
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: latest-darn-outcome-stat-error-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: darndeps-latest-outcome-closure-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised three LOW findings against `internal/loomcli/darnwiring.go`.
F1 maps to `latest-darn-outcome-goto-skipped-entries`, F2 to `latest-darn-outcome-stat-error-untested`, F3 to `darndeps-latest-outcome-closure-untested`.
Each fix text fixes its result, so all are mechanical.

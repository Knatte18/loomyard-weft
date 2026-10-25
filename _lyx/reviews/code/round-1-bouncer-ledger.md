---
round: 1
ledger:
  - key: atorbelow-no-cycle-guard
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: orch-row-docs-overview-stale
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: beam-status-run-cells-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: orch-order-claim-false
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised four findings.
F1 (`atorbelow-no-cycle-guard`) is the only gating one.
F2 to F4 are LOW doc, test and comment corrections.

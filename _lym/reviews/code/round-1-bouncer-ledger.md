---
round: 1
ledger:
  - key: existing-baseline-guard-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: report-wrong-scope-provenance
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: verify-stencil-baseline-expectation
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised three LOW findings: F1 (untested `baseline.md` guard in `settleConfiguredScope`), F2 (generated report names the drift config as the scope's source), F3 (Drift-Verify stencil describes `baseline.md` as an agent's expectation).
None gates, so the loop converges.

---
round: 1
ledger:
  - key: drift-verdict-seam-exempt-from-nil-guard
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: summary-section-only-when-describe-runs
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: merge-message-section-untested
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: landing-commit-message-beyond-entry
    rounds: [1]
    status: open
    class: decision
    severity: LOW
    fix: design-wrong
  - key: added-paths-without-test-row
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: compare-recorded-assertion-cannot-tell-commits-apart
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised six findings, none recurring.
The only design-class finding is LOW, so nothing gates and the loop converges.
The two decision-class findings are deferred to the run's parent.

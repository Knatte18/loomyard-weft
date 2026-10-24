---
round: 1
ledger:
  - key: webster-fingerprint-includes-lens-reports
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: specsdir-fill-untested-in-burler-geometry-builders
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: engine-review-specsdir-forwarding-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: solo-reviewer-prompt-text-changed
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised four findings: F1 to F4 in the review, mapped in order to the keys above.
Only the SpecsDir geometry test gap is gating.
Every fix text names the exact edit, so all four are mechanical.

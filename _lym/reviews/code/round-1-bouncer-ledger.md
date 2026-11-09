---
round: 1
ledger:
  - key: cancelled-step-kept-as-complete
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: stale-hash-cover-exclusion-unpinned
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: runtest-wiring-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: lym-build-and-corrupt-map-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: spawn-scan-drops-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: mid-package-budget-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: retry-error-line-and-match-false-cases-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: failed-report-write-and-slot-hold-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised eight design findings.
F1 (cancelled-step-kept-as-complete) is a one-line retry rule fix; F2 to F8 are missing test rows.
Keys map to review IDs in order: F1 to F8.

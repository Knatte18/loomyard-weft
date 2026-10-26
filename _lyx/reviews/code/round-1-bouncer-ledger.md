---
round: 1
ledger:
  - key: rename-leaves-test-misses-external-test-package
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: failed-report-handback-exclusion-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: exact-rename-exemption-unpinned-in-engine
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: rename-leaves-test-none-language-row-missing
    rounds: [1]
    status: open
    class: design
    severity: NIT
    fix: mechanical
---

Round 1 is the first round; there is no earlier ledger.
F1 (the external test package's `<dir>_test` unit never matching a renamed symbol's unit) is the only gating finding.
F2, F3 and F4 are test-coverage gaps with fixes the review text already fixes.

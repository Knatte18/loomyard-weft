---
round: 1
ledger:
  - key: no-test-seam-and-bug-label-repeated-literals
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: non-go-hub-red-first-exits-zero
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: judge-red-first-drops-named-failure-in-mixed-report
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: base-run-keeps-non-go-inputs
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: gate-covers-only-darn-unrecorded-narrowing
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: mechanical
  - key: verify-site-tripwire-and-docs-miss-new-site
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: rubric-missing-trailer-clause-over-reaches
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: merge-danger-blast-radius-too-narrow
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: entry-labels-has-no-behaviour-test
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: changed-tests-at-package-selection-not-pinned
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: red-first-gate-missing-rows
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: red-first-command-gate-key-not-asserted
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: run-pattern-union-runs-same-named-tests
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
---

Round 1 found 13 findings.
The gating ones are the repeated `No test seam` and `bug` literals (F1), the non-Go hub that exits 0 as if a test went red (F2),
the named failure lost in a mixed report (F3), the base run keeping HEAD's non-Go inputs (F4) and the untested `entryLabels` (F9).
F5 is a decision finding deferred to the run's parent.
F6 to F8 are consistency findings on docs, the rubric clause and the merge-danger section.
F10 to F13 are LOW test and edge-case findings.

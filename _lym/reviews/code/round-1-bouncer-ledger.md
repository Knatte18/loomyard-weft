---
round: 1
ledger:
  - key: cleanup-help-spells-records-namespace
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: catchup-layout-error-misreported-unreachable
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: o2-layout-reads-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: report-only-listing-fails-cleanup
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: killed-seed-clone-names-failing-join
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: layout-error-swallowed-into-two-url-command
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: new-behaviours-without-test-row
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: summary-claims-single-repo-in-o1
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: summary-merge-danger-incomplete
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: reset-rerun-drops-force-bootstrap
    rounds: [1]
    status: open
    class: design
    severity: NIT
    fix: mechanical
---

Round 1 raised ten findings, all open: F1 to F10 in the order above.
F1 is BLOCKING and F2 and F3 are MEDIUM design findings, so they gate.
F2 and F4 change an error contract or the envelope shape, so their fixes need a fresh review.
The rest are local corrections the finding text already fixes.

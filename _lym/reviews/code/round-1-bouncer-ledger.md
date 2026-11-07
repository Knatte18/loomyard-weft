---
round: 1
ledger:
  - key: settings-render-keep-takeout-untested
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: settings-all-invalid-entries-strip-keys
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: settings-record-removed-before-file
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: settings-reinterpretation-no-reconcile-test
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
---

Round 1 raised four findings, all in the heddle settings render and its fabric reinterpretation.
None is gating, so the loop converges.
The previous ledger was `(none)`, so nothing carries forward.

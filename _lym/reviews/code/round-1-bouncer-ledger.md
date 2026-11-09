---
round: 1
ledger:
  - key: ended-early-causes-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: darn-write-stop-on-fail-unpinned
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: budget-wiring-unpinned
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: orphaned-doc-comment-gateendedearly
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: untested-failure-branches
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: budget-notice-latest-failure-only
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
---

Round 1 raised six findings, none seen before.
F1 to F3 are test-coverage gaps in the gate-faults fix and gate the loop.
F4 is a misplaced doc comment in `internal/shuttleengine/wait.go`.
F5 is a bundle of untested failure branches.
F6 asks the budget notice to list every failed run to date instead of the latest.

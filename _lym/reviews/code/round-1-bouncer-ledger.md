---
round: 1
ledger:
  - key: pins-left-uncommitted-after-failed-seed-commit
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: notice-overclaims-stencil-diff
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: start-stencil-check-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: seed-commit-of-pins-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: parked-driver-wake-check-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: failed-parent-post-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: changedpins-ordering-and-unreadable-copy-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: start-resends-notice-on-noop
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: creel-post-under-bootstrap-lock
    rounds: [1]
    status: open
    class: design
    severity: NIT
    fix: mechanical
---

Round 1 raised nine findings: F1 to F9 map in order to the keys above.
Three are gating (F1, F3, F4).
F8 changes which start paths warn, a behavior contract, so it needs a fresh review.

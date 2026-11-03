---
round: 1
ledger:
  - key: dead-holder-lock-clear-unrecorded
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: landing-push-retry-uninterruptible
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: empty-holder-record-reads-as-live
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: dead-pid-clause-claims-unknown-cause
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: holder-record-sites-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: holder-record-failure-paths-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 is the first round, so there is no earlier ledger to carry forward.
The six findings map one to one to F1 through F6 of the round 1 review.
None is gating, so the loop converges with all six recorded as open.

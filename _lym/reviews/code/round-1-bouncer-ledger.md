---
round: 1
ledger:
  - key: exit-record-loses-unmeasured-jobs-on-error
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: no-test-single-slot-hub-pass
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: retake-after-yield-unpinned
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: no-test-head-read-on-slot-wait-wake
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: no-test-named-call-waits-on-build-lock
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: empty-planned-head-on-early-refusal
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: no-test-inherited-slot-call
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: no-test-signal-keeps-finished-rounds
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: no-test-upkeep-cap-default
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: no-test-failed-upkeep-registration
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: stale-exit-file-name-comment
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised eleven findings, all open.
F1 is a correctness gap: an error or signal ends a pass without counting its unmeasured jobs as queued.
F2 to F10 are coverage gaps for behavior the diff adds, and F6 is an early-refusal record with an empty planned HEAD.
F11 is a stale comment.
Every fix is a local correction its finding text already fixes.

---
round: 1
ledger:
  - key: reset-unchanged-state-lands-no-transition
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: watched-notice-premise-contradicts-batten-hold
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: judged-flag-outlives-its-stop
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: goto-transition-narrowing-unrecorded
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: design-wrong
  - key: pattern-run-fsm-misses-reset-transition
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: plan-checks-spec-lacks-overview-frame-edited
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: no-card-index-only-edit-passing-row
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: wirehub-runid-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: gate-silent-on-records-without-frame-hash
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: unreadable-overview-error-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: unborn-warp-trailers-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: drivenotices-missing-keep-mark
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised twelve findings, none carried from an earlier round.
The core defect is that `webster reset` lands no Run-Transition commit when the records bytes do not change, which the incident's own shape produces.
Around it sit a false premise about batten's hold, a `judged` flag that outlives its stop, an unrecorded narrowing for `goto`, two stale documents, and four test gaps.

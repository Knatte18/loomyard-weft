---
round: 1
ledger:
  - key: accepted-profile-wiring-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: lapse-drops-every-acceptance
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: accepted-capture-timeout-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: reportengine-doc-omits-drift-recheck
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: drift-template-enabled-comment-stale
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: tapestry-header-names-old-perch-exit
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised six findings, all open.
The gating one is the missing test for the accepted-profile seams, `CompareAccepted` and `CaptureAccepted`, a mechanical fix.
The lapse-all behavior on drift is a tradeoff whose fix changes the approach or its documented contract, so it is `design-wrong`.
The other four are local test or wording corrections.

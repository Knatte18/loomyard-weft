---
round: 1
ledger:
  - key: d1-coverage-read-before-idle-wait
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: d6-orch-guid-shape
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: d1-setpause-holds-orch-start
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d4-verify-starting-under-pause-not-held
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d4-sigkill-escalation-not-from-waitdelay
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d4-cancel-change-reaches-other-verifyrun-callers
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: design-wrong
  - key: d8-failed-guid-lookup-strands-master
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d1-fail-open-states-unbounded
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: d1-sendconditioned-bypasses-sendverified
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d7-reread-does-not-exclude-resume
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: d3-expired-finalize-runs-gate-under-cover
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d4-activity-readers-misread-paused-verify
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: plan-note-cancel-path-missing-at-two-sites
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d2-deferred-prompt-not-run-start-send
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: acceptance-criteria-miss-negative-bounds
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

First round; no earlier ledger.
Fifteen findings from the review, F1 to F15, in ledger order.
The BLOCKING one is D1's coverage read placed before the idle wait.
The MEDIUM design ones are the orch GUID shape in D6, `SetPause` arming `lyx orch start`'s wait, D4's unheld Verify start and its SIGKILL escalation, and D8's failed GUID lookup.
D4's cancel change reaching other `verifyrun.Run` callers is MEDIUM but `consistency`.
The rest are LOW.

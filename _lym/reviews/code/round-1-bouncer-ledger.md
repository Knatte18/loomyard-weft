---
round: 1
ledger:
  - key: covered-paused-run-no-pass-or-notice
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: dobbyruns-reachedparent-read-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: dobbyengine-doc-stale-first-bullet
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: reached-parent-warn-every-tick
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: liftpending-cleanup-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: goto-leaves-paused-run-resumed
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: testplanpass-missing-paused-rows
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: batten-pause-pointers-stale
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised eight findings, none recurring.
The gating one is the missing test for the `DobbyRuns` reached-parent read.
The covered-run narrowing (F1) is an unrecorded parent correction and goes to the run's parent.
The `goto` finding (F6) may change the contract, so it is marked design-wrong.

---
round: 1
ledger:
  - key: overview-dobby-section-in-task-pair
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: overview-poll-takes-dobby-and-driver-locks
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: dobbytasks-change-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: refusal-clearing-unasserted
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: view-done-run-in-flight-takes-no-slot
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: full-no-reopen-log-name-unasserted
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: overviewcmd-stale-prime-only-wording
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised seven findings, all still open.
F1 is BLOCKING: the overview draws a dobby section in a task pair's frame.
F2 is the only design-wrong fix: the pane's lock probes can make a starting dobby or a hand-run `lyx batten run` refuse.
F3 and F4 are missing test coverage for `DobbyTasks` and for refusal clearing.
F5 and F6 are LOW: slot counting for a done run with a pass in flight, and an unasserted log name.
F7 is a stale comment and `Short` text.

---
round: 1
ledger:
  - key: owed-push-still-writes-holding-record
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: prime-pusher-reads-no-batten-takeover
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: paused-pair-pusher-keeps-pushing
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: pusher-runproc-owner-is-slug
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: handover-pair-removal-flag-open
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: driver-stop-record-pushes-not-in-scope
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: design-wrong
  - key: code-tip-window-reads-as-wait
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: refused-prime-push-stops-every-batten-run
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: teardown-unbounded-wait-under-prime-lock
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d1-no-wait-vs-d5-teardown-wait
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: d4-restates-existing-behaviour
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: no-upstream-branch-no-dobby-restart
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: sleeping-pusher-lock-wording
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d3-start-rule-omits-refused-record
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: spawnstatuspush-not-unused
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 is the first round, so no earlier ledger exists.
The three BLOCKING findings are F1 (`owed-push-still-writes-holding-record`), F2 (`prime-pusher-reads-no-batten-takeover`) and F3 (`paused-pair-pusher-keeps-pushing`).
All three show the pusher design breaking an existing guard or record rule, and each needs a changed decision.
The MEDIUM design findings F4 to F9 cover owner keys, hand-over flags, scope of driver-stop pushes, the code-tip window, hub-wide refusal effect and the teardown wait under the prime lock.
The remaining findings are local wording or consistency corrections.

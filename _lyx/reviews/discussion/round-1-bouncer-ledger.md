---
round: 1
ledger:
  - key: d9-duplicates-reed-reap
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: d5-gate-wait-unreadable
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: end-before-loop-kills-merge-step
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
    fix: design-wrong
  - key: remote-git-children-escape-end
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: reed-down-readings-undefined
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: d7-misses-batten-owner-probe
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: stale-running-batten-record
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d8-deleted-cwd-reaps-live-pair
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: driver-lock-recheck-callers
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: pattern-text-vs-d5-skip-readings
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d10-sixty-minute-evidence
    rounds: [1]
    status: open
    class: decision
    severity: LOW
    fix: mechanical
  - key: reap-signal-unstated
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d11-open-risk-shell-end
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: windows-remove-paused-refuses
    rounds: [1]
    status: open
    class: decision
    severity: LOW
    fix: mechanical
  - key: support-log-stale-d-numbers
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: lift-fact-names-wrong-function
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised sixteen findings, all open.
The gating cluster is the process-end design: D9's false premise about reed's reap, D5's unreadable gate wait, D4's ordering against the run-lock wait, remote git children outside End's group, and `reed down`'s undefined readings.
The MEDIUM design findings concern batten's owner probe, a stale running batten record, and D8's deleted-directory rule.
The rest are local corrections the finding text already fixes.

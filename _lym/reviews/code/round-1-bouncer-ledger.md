---
round: 1
ledger:
  - key: turn-on-remote-control-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: rc-click-on-already-on-session
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: launch-branches-no-test-rows
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: rc-bar-fallbacks-no-test-rows
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: duplicate-orch-strands-misreported
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: adopted-strand-loses-lit-state
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: record-write-failure-reported-as-failed-click
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: codex-orch-draws-rc-button
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: design-wrong
  - key: orch-group-help-stale
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: contained-reed-tests-need-keep-marks
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised ten findings, none recurring.
The gating one is `turn-on-remote-control-untested` (F1).
`rc-click-on-already-on-session` (F2) needs Claude Code's behaviour verified before its fix is known.
The rest are LOW: missing test rows (F3, F4), duplicate-strand and adopt handling (F5, F6), write-failure reporting (F7), the Codex button (F8), stale help (F9) and keep marks (F10).

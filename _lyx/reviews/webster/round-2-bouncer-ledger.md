---
round: 2
ledger:
  - key: selvage-bottom-most-test-assumes-shared-window
    rounds: [2]
    status: open
    class: scope
    severity: BLOCKING
    fix: mechanical
  - key: attach-argv-degraded-test-stale-argv
    rounds: [2]
    status: open
    class: scope
    severity: BLOCKING
    fix: mechanical
  - key: bar-test-hardcoded-window-index
    rounds: [2]
    status: open
    class: scope
    severity: BLOCKING
    fix: mechanical
  - key: alt-step-test-ignores-own-windows
    rounds: [2]
    status: open
    class: scope
    severity: BLOCKING
    fix: mechanical
  - key: status-bar-click-test-zooms-pane-outside-shell-window
    rounds: [2]
    status: open
    class: scope
    severity: BLOCKING
    fix: mechanical
  - key: operator-active-test-attaches-wrong-strand
    rounds: [2]
    status: open
    class: scope
    severity: BLOCKING
    fix: mechanical
  - key: paste-test-races-sink-stty
    rounds: [2]
    status: open
    class: scope
    severity: BLOCKING
    fix: mechanical
  - key: dead-pane-move-test-relaunch-exits
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
---

Round 1 raised nothing.
Round 2 raised eight BLOCKING findings, all in tests that predate the own-window placement of strands, plus one that may hide a `Resume` defect for a moved dead pane.
All eight keys are new this round.

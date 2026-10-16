---
round: 3
ledger:
  - key: selvage-bottom-most-test-assumes-shared-window
    rounds: [2]
    status: resolved
  - key: attach-argv-degraded-test-stale-argv
    rounds: [2]
    status: resolved
  - key: bar-test-hardcoded-window-index
    rounds: [2]
    status: resolved
  - key: alt-step-test-ignores-own-windows
    rounds: [2]
    status: resolved
  - key: status-bar-click-test-zooms-pane-outside-shell-window
    rounds: [2]
    status: resolved
  - key: operator-active-test-attaches-wrong-strand
    rounds: [2]
    status: resolved
  - key: paste-test-races-sink-stty
    rounds: [2]
    status: resolved
  - key: dead-pane-move-test-relaunch-exits
    rounds: [2]
    status: resolved
---

Round 1 raised nothing.
Round 2 raised eight BLOCKING findings, all in tests that predate the own-window placement of strands.
Round 3 raised nothing, and its review confirmed the eight named tests pass under the tmux gate, so all eight keys are resolved.

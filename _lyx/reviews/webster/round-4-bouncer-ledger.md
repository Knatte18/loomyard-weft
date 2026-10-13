---
round: 4
ledger:
  - key: own-window-relaunch-leaves-dead-pane
    rounds: [1]
    status: resolved
  - key: placement-constants-mixed-iota
    rounds: [1]
    status: resolved
  - key: claudeengine-test-budget-row-short
    rounds: [2]
    status: resolved
  - key: unseen-clear-hook-bare-session-target
    rounds: [4]
    status: open
    class: design
    severity: BLOCKING
  - key: openwindow-options-hit-same-named-own-window
    rounds: [4]
    status: open
    class: design
    severity: BLOCKING
  - key: pane-gone-assertion-display-message
    rounds: [4]
    status: open
    class: design
    severity: BLOCKING
---

Rounds 1 to 3 closed the relaunch dead-pane, mixed-`iota` placement constants and `claudeengine` test-budget findings.
Round 4 came from the publish failure and raised three BLOCKING design findings, each tied to one failing test: the `set-hook` target needing the `=<session>:` form (F1), `OpenWindow` chaining options onto a name an own window can share (F2), and a pane-gone assertion that `display-message` cannot satisfy under tmux 3.6 (F3).

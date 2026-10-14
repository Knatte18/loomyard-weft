---
round: 1
ledger:
  - key: row-placement-departs-from-board-entry
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: stale-batten-paragraph-overview
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: terminal-window-envelope-text
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: seedtask-keybindings-unpinned
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised four findings, none gating.
F1 (row placement) is for the run's parent to record or refuse.
F2 and F3 are stale doc and text corrections; F4 asks for one assertion in an existing `ideengine` test.

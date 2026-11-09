---
round: 1
ledger:
  - key: status-window-not-titled-through-shared-function
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: design-wrong
  - key: terminalwindowtitle-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: unreadable-board-loses-beam-label
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: testwindowtitles-misses-branches
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: stale-batten-title-test-name-and-comments
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised five findings (F1 to F5), all open.
F1 asks whether the loom status window falls under the shared title function.
F2 is the only gating one: the real `terminalWindowTitle` is never exercised.
F3 to F5 are LOW test-coverage, doc and naming gaps.

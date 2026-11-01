---
round: 1
ledger:
  - key: prime-spawn-window-dir-is-file
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: window-wait-unpinned
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: window-command-timeout-untestable
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: opentask-and-failed-launch-untested
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: place-terminal-window-never-run
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: blank-window-command-unpinned
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: unrelated-config-help-rewording
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: vacuous-unset-marker-check
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised eight findings, all open.
The gating ones are the prime spawn running the window command in the workspace file path (F1) and the untestable, unpinned wait and timeout in `desktophook` (F2, F3).
The rest are test-coverage gaps, one out-of-scope help-text edit and one vacuous assertion.

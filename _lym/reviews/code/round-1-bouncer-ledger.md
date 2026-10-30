---
round: 1
ledger:
  - key: wake-gates-before-status-read
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: no-path-removes-record-before-session-stop
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: session-unrecorded-on-failed-record-write
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: gone-worktree-warn-lacks-task-branch
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: hold-while-covered-doc-contradicts-warn
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: open-continue-clean-ungated-row-unpinned
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 (first round, no prior ledger).
F1 maps to `wake-gates-before-status-read`: the only gating finding, since a pull request merged at an ungated head is recorded as gated and ends Done.
F2 and F3 are LOW design findings about sessions left live (`no-path-removes-record-before-session-stop`, `session-unrecorded-on-failed-record-write`).
F4, F5 and F6 are LOW consistency findings: the gone-worktree Warn, the `holdWhileCovered` doc, and a missing wait-test row.

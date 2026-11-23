---
round: 1
ledger:
  - key: done-projects-offered-in-dropdown
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: done-project-nonroot-parent-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: list-text-unchanged-goal-narrowed
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: mechanical
---

Round 1 raised three MEDIUM findings.
The viewer's project dropdown now lists done projects that every write refuses (F1).
The fixture's only done project hangs under the root, so a hard-coded root parent passes every test (F2).
The `lym board list --text` listing the Goal names is left unchanged without a recorded parent decision (F3, deferred to the parent).
Each fix is a local correction its finding text already fixes.

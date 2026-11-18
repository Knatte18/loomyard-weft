---
round: 2
ledger:
  - key: d3-window-name-source-unsettled
    rounds: [1]
    status: resolved
  - key: d1-role-held-refusal-untyped
    rounds: [1]
    status: resolved
  - key: d1-add-races-concurrent-loom-start
    rounds: [1]
    status: resolved
  - key: d1-failure-paths-lose-checklist
    rounds: [1]
    status: resolved
  - key: parent-commit-7c23591-conflict
    rounds: [1]
    status: resolved
  - key: overview-no-lock-constraint-contradiction
    rounds: [1]
    status: resolved
  - key: d1-same-worktree-premise-evidence
    rounds: [1]
    status: resolved
  - key: no-criterion-failed-status-sidecar-write
    rounds: [1]
    status: resolved
  - key: d2-criterion-structural-clause
    rounds: [1]
    status: resolved
  - key: open-risk-q6-settled-by-added-entries
    rounds: [2]
    status: open
    class: consistency
    severity: BLOCKING
    fix: mechanical
---

Round 1's nine findings (F1 to F9) are all resolved in round 2's prior list.
F5, the parent-commit question, is settled by the parent's two added entries.
Its leftover is a new round 2 finding, F10: the Open risks bullet on commit 7c23591 and the Q6 line in the support log still read as open.
The fix is mechanical: drop the bullet and mark Q6 settled.

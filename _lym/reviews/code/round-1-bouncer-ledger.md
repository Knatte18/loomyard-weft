---
round: 1
ledger:
  - key: render-fanout-staging-and-out-of-hub-worktrees
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: no-render-and-override-fanout-unpinned
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: failed-worktree-branch-and-render-warnings-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: failed-worktree-render-way-forward-inconsistent
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: render-warnings-undocumented
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: sync-after-render-order-unpinned
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: prime-status-message-misleading
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 is the first round; there is no earlier ledger.
Keys map to F1 through F7 of the review in order.
The three MEDIUM design findings gate; the rest are local corrections the review text already fixes.

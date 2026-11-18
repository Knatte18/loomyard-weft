---
round: 1
ledger:
  - key: d3-window-name-source-unsettled
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: d1-role-held-refusal-untyped
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d1-add-races-concurrent-loom-start
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d1-failure-paths-lose-checklist
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: parent-commit-7c23591-conflict
    rounds: [1]
    status: open
    class: decision
    severity: BLOCKING
    fix: design-wrong
  - key: overview-no-lock-constraint-contradiction
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d1-same-worktree-premise-evidence
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: no-criterion-failed-status-sidecar-write
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d2-criterion-structural-clause
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised nine findings.
D3's window-name source (F1) blocks and, with the D1 findings F2, F3 and F4, gates the loop.
F5 asks the run's parent whether to align with commit 7c23591; it is not fixable by the fixer.
F6 to F9 are local corrections; F6 depends on the reader F1 settles.

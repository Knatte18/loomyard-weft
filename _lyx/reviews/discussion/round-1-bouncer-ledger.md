---
round: 1
ledger:
  - key: prime-spawn-criterion-vs-degrade-path
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: in-place-write-truncation-vs-writes-nothing
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: lock-placement-no-board-hub
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: retired-artifact-warn-trigger-undefined
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: nonexistent-hub-detail-fields
    rounds: [1]
    status: open
    class: decision
    severity: LOW
    fix: design-wrong
  - key: d2-skip-unresolvable-worktree
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: d4-bound-overstates-self-healing
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: reconcile-sweep-regenerate-order
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d6-retires-helpers-d7-reuses
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: directory-at-retired-workspace-path
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: prime-branch-change-stale-display-name
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: unchanged-folder-list-criterion-narrow
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised twelve findings, none carried from an earlier round.
Two are gating: the truncating in-place write that contradicts the writes-nothing claim and the D4 Bound, and the unplaced lock on a hub with no `_board`.
The other two MEDIUM findings are consistency: the prime-spawn criterion against Spawn's degrade path, and the undefined retired-artifact Warn trigger.
The rest are LOW or NIT wording and bound gaps in D2, D4, D6 and D7.

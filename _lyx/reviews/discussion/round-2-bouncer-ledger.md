---
round: 2
ledger:
  - key: prime-spawn-criterion-vs-degrade-path
    rounds: [1]
    status: resolved
  - key: in-place-write-truncation-vs-writes-nothing
    rounds: [1]
    status: resolved
  - key: lock-placement-no-board-hub
    rounds: [1]
    status: resolved
  - key: retired-artifact-warn-trigger-undefined
    rounds: [1]
    status: resolved
  - key: nonexistent-hub-detail-fields
    rounds: [1]
    status: resolved
  - key: d2-skip-unresolvable-worktree
    rounds: [1]
    status: resolved
  - key: d4-bound-overstates-self-healing
    rounds: [1]
    status: resolved
  - key: reconcile-sweep-regenerate-order
    rounds: [1]
    status: resolved
  - key: d6-retires-helpers-d7-reuses
    rounds: [1]
    status: resolved
  - key: directory-at-retired-workspace-path
    rounds: [1]
    status: resolved
  - key: prime-branch-change-stale-display-name
    rounds: [1]
    status: resolved
  - key: unchanged-folder-list-criterion-narrow
    rounds: [1]
    status: resolved
  - key: portal-restored-launcher-only-restore-unreported
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: workspace-file-overwrites-vscode-edits
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: writes-nothing-vs-lock-file
    rounds: [2]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised twelve findings, two of them gating.
Round 2 re-raised none of them; its focus outcomes confirm the D3, D4, D5 and D7 points hold, so all twelve are resolved.
Round 2 raised three new findings.
The MEDIUM one is that retiring `portal_restored` leaves a launcher-only restore unreported.
The LOW one is that regeneration overwrites edits made in VS Code.
The NIT is that "writes nothing" ignores the lock directory and file.

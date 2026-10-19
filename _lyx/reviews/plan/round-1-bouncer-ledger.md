---
round: 1
ledger:
  - key: card1-portals-assert-before-retirement
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: hub-workspace-folders-admit-outside-worktrees
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: portals-chain-removal-bypasses-gate
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: vscode-package-doc-stale
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: sweep-refusal-path-unspecified
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: concurrent-regeneration-row-blind-to-lock
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: mark-read-failure-mislabelled-wiring-guard
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: retired-file-removal-recursive-after-lstat
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: truncated-prefix-row-duplicates-coverage
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: portallauncher-test-header-stale
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: spawn-failed-write-row-reaches-read-branch
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: readonly-restore-row-posix-only
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: row5-global-hook-not-serial
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised 13 findings against the plan, none carried from an earlier round.
Three are BLOCKING `design` findings: a card 1 assertion that depends on card 4, a folder listing that admits worktrees outside the hub, and a `_portals` chain removal that skips the destruction gate's checks.
The remaining ten are `scope` or `consistency`: a stale package doc, an unspecified sweep error path, and test rows that cannot reach or detect what they claim to.

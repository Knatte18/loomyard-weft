---
round: 2
ledger:
  - key: card1-portals-assert-before-retirement
    rounds: [1]
    status: resolved
  - key: hub-workspace-folders-admit-outside-worktrees
    rounds: [1]
    status: resolved
  - key: portals-chain-removal-bypasses-gate
    rounds: [1]
    status: resolved
  - key: vscode-package-doc-stale
    rounds: [1]
    status: resolved
  - key: sweep-refusal-path-unspecified
    rounds: [1]
    status: resolved
  - key: concurrent-regeneration-row-blind-to-lock
    rounds: [1]
    status: resolved
  - key: mark-read-failure-mislabelled-wiring-guard
    rounds: [1]
    status: resolved
  - key: retired-file-removal-recursive-after-lstat
    rounds: [1]
    status: resolved
  - key: truncated-prefix-row-duplicates-coverage
    rounds: [1]
    status: resolved
  - key: portallauncher-test-header-stale
    rounds: [1]
    status: resolved
  - key: spawn-failed-write-row-reaches-read-branch
    rounds: [1]
    status: resolved
  - key: readonly-restore-row-posix-only
    rounds: [1]
    status: resolved
  - key: row5-global-hook-not-serial
    rounds: [1]
    status: resolved
  - key: card1-row7-folder-change-via-verb
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: card2-row5-passes-without-rollback-regeneration
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: applystalepairownership-doc-names-portal-debris
    rounds: [2]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: card5-portal-links-planted-with-os-symlink
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised 13 findings, three of them BLOCKING `design`.
Round 2's review raised none of those keys again, so all 13 are carried as resolved.
Round 2 raised four new findings, none BLOCKING: a card 1 test row whose folder-list change passes at card 1 but breaks at card 2's gate, a card 2 rollback row that cannot fail, one portal-debris doc comment outside every card, and card 5 planting portal links with `os.Symlink` instead of `fslink.CreateDirLink`.
Each fix is text-only and fixed by the finding itself.

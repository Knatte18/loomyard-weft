---
round: 1
ledger:
  - key: heddlecli-lacks-runcli
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
    fix: mechanical
  - key: runproc-allowlist-misses-lymcwd
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
    fix: mechanical
  - key: new-test-packages-no-testmain
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: boltdirname-guard-no-lym-guard-marker
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
    fix: mechanical
  - key: run-ledger-decision-departs-d7
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: heddle-cli-departs-cliwire-constraint
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: bolt-paths-outside-fabric-use-boarddir
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: fastforwardpair-no-took-in
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: design-wrong
  - key: reconcile-reseed-row-unreachable
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: heddle-set-no-attack-surface-bound
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: no-board-file-guard-scans-two-files
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: config-load-order-unstated
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: drivable-worktree-refusal-no-row
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: restorereplaytip-message-stale
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: pattern-background-files-board-name
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: config-go-header-stale
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: livestate-doc-test-board-literal
    rounds: [1]
    status: open
    class: decision
    severity: LOW
    fix: mechanical
  - key: excludewayforward-on-wrong-card
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: entrymerge-table-duplicates-fabricreplay
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: card6-import-cycle-packages-unnamed
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 is the first round; there was no prior ledger.
The key-to-finding mapping, in review order: F1 to F20.

BLOCKING: F1 (`RunCLI`/`RunCLIIn` missing on heddlecli), F2 (runproc import allowlist), F3 (no `TestMain` in three new test packages), F4 (missing `//lym:guard` marker).
Design findings at MEDIUM: F7 (Bolt paths outside fabric), F10 (attack-surface bound on `lym heddle set`/`remove`).
Decision-class: F5 and F6 need parent records in the decision record; F17 is a local fix.
The rest are LOW or local consistency corrections.

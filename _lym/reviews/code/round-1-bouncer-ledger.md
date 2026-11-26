---
round: 1
ledger:
  - key: absent-beam-worktree-shows-parent-values
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: beam-pr-link-may-be-spent
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: prime-of-board-bypasses-fabric-lister
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: config-doc-stale-reader-list
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: untested-beam-and-unknown-slug-sources
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: untested-project-file-mode-line
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: untested-beam-filter-and-reader-failure
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised one MEDIUM and the rest LOW.
The MEDIUM concerns what the board render shows for a beam whose worktree is on another machine.
The LOW findings cover the Beam PR link, the worktree lister, a stale doc line and test gaps.

---
round: 1
ledger:
  - key: wiredarn-verify-gate-told-scoped-command
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: treescopedcommand-nested-trees-drop-target
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: moduleguardtests-silent-skip-outside-module
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: guard-name-dedup-unpinned
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: root-package-guard-branch-uncovered
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: newverifygate-doc-stale-wholediff
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: darnengine-doc-opening-contradicts-verify
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised seven findings, none BLOCKING.
Two are gating: `wireDarn` hands the verify gate the per-call scoped command where Card 7 says the hub's whole verify, and `TreeScopedCommand` credits a path only to the first sorted tree, so nested trees lose the specific tree's target.
The rest are an unreachable silent skip in `ModuleGuardTests`, two coverage gaps in the guard path, and two stale doc statements.

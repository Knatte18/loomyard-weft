---
round: 1
ledger:
  - key: empty-revision-takes-create-path
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: cards-7-9-missing-bound
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: keepalive-shutdown-order
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: opener-cmd-wait-unrecorded
    rounds: [1]
    status: open
    class: decision
    severity: LOW
    fix: mechanical
  - key: serve-running-branch-untested
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: expecting-revision-duplicate-row
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: stale-render-replay-comments
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: overview-package-tree-boardserve
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: runproc-doc-extends-closed-list
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: shared-decisions-promote-misdescribed
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised ten findings, none recurring.
The gating one is the empty-revision path: card 2 reads an empty revision as a create, and card 4 lets non-create ops pass it through.
The keep-alive shutdown order in card 5 is a design finding at LOW.
The rest are local corrections: missing bounds on cards 7 and 9, an unrecorded `cmd.Wait` substitution, one untested `serve` branch, a duplicate test row, and stale comments or docs.

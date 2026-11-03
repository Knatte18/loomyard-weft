---
round: 1
ledger:
  - key: add-rollback-leftover-finish-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: sibling-registered-guard-unpinned
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: sibling-finish-failure-after-git-failure-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: list-failure-licenses-recursive-delete
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: finish-failure-no-way-forward
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: summary-unverified-cause
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised six findings (F1 to F6, in the order above); there was no earlier ledger.
F1 and F2 gate: test coverage missing for the Add rollback's leftover finish and for the sibling side's registered-worktree guard.
F3 to F6 are local corrections the review text already fixes.

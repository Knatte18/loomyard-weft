---
round: 1
ledger:
  - key: weft-lock-dir-add-rows-vacuous
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
  - key: weft-switch-failure-leaves-created-branch
    rounds: [1]
    status: open
    class: scope
    severity: LOW
  - key: add-rollback-rows-in-preflight-test-file
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
---

Round 1 is the first round, so there is no prior ledger to carry forward.

- `weft-lock-dir-add-rows-vacuous` (F1): card 1's new add-path test passes before its own Add edit, and the paths D1 changes (adopted weft with an origin record, dormant raw fork) carry no `.weft/` assertion.
- `weft-switch-failure-leaves-created-branch` (F2): a failing weft switch after the resolver created the local branch leaves that branch behind, because `Checkout` passes no created branch to `rollbackSwitch`.
- `add-rollback-rows-in-preflight-test-file` (F3): card 4's rollback rows belong in `add_rollback_adopt_test.go`, which holds the existing adopted-branch rollback tests and the portal-blocker injection.

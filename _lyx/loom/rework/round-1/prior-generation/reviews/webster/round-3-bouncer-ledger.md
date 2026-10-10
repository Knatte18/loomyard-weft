---
round: 3
ledger:
  - key: log-level-env-reset-by-root-hook
    rounds: [1]
    status: resolved
  - key: boot-refusals-unreached-at-planned-site
    rounds: [1]
    status: resolved
  - key: cmd-lyx-test-budget-row-not-raised
    rounds: [3]
    status: open
    class: scope
    severity: BLOCKING
---

Round 1 raised two findings, the root hook resetting the level `LYX_LOG_LEVEL` set and the boot refusals shadowed at their planned site; round 2 found both fixed.
Round 3 raised one new finding: the root-hook test added by the round 1 fix pushed `cmd/lyx` to 74 top-level tests, but the merge-in brought the budget row at 73, so Publish verify fails `TestTestBudget_PackagesStayWithinBudget`.
The fix is to raise the `cmd/lyx` row in `cmd/lyx/testdata/test-budget.yaml` to 74.

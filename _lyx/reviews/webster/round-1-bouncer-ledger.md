---
round: 1
ledger:
  - key: plan-card-write-pattern-md-suffix
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised one finding, F1: the plan-card write pattern in `tools/forkaudit/plan-writer.jq` lacks the `.md` anchor card 6 names.
The fix is local and its text is fixed by the finding, so it is mechanical.
There was no previous ledger to carry forward.

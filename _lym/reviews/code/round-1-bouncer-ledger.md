---
round: 1
ledger:
  - key: summary-denies-commit-after-merge
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
---

Round 1 is the first round and has no prior ledger.
F1 maps to `summary-denies-commit-after-merge`: the change description says no commit follows the merge, but `382a591f` does.
The fix is a local text correction the finding already spells out.

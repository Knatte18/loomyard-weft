---
round: 1
ledger:
  - key: bug-record-rule-unscoped
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
---

Round 1 raised one finding, F1, mapped to `bug-record-rule-unscoped`.
The new bug-entry record rule in the discussion spec is unscoped, so darn Record-Write records, which no writer can fill, break it.
The fix is a local rewording of the subsection's opening line to scope it to the Discussion-Write lead's records.
There was no previous ledger to carry forward.

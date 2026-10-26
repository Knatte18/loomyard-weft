---
round: 1
ledger:
  - key: checklist-box-precedence-untested
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
---

Round 1 raised one finding (F1): no test row pins that a skipped row inside a visited review segment leaves the heading box unchanged, nor that a current producer with a skip entry keeps its state box.
It is LOW and non-gating; the fix is two added test rows.

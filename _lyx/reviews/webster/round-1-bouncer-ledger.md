---
round: 1
ledger:
  - key: retired-fan-review-key-in-tests
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised one finding, F1: three tests still use the retired `fan_review` key as the example model key of a fanned reviewer.
It is a NIT of class consistency whose fix is a local literal swap, so it is mechanical.
There was no previous ledger to carry forward.

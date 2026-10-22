---
round: 1
ledger:
  - key: webster-template-trailing-newline
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised one finding, F1: the embedded webster `template.yaml` ended without a final newline.
It is a local whitespace correction.
There was no previous ledger to carry forward.

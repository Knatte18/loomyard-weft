---
round: 1
ledger:
  - key: readtaskpatch-linked-scratch-untested
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
---

Round 1 raised one finding, F1: no test runs `ReadTaskPatch` over a warp whose root `.scratch` is the fabric-wired link.
The review judged the interaction correct by reading, so the finding asks only for a pinning test row.

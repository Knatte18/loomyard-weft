---
round: 1
ledger:
  - key: shedverbs-fixture-grace-unlicensed
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: redundant-golden-covered-contains-row
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 is the first round, so there is no prior ledger to carry forward.
F1 maps to `shedverbs-fixture-grace-unlicensed`: a test-only flake fix outside the board entry's scope, deferred to the parent.
F2 maps to `redundant-golden-covered-contains-row`: a `Contains` row the golden comparison already pins.

---
round: 1
ledger:
  - key: seed-call-asks-discarded-exclusions
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
  - key: exclusion-union-undefined-across-fan-switch
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: lens-texts-restate-rubrics
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
  - key: acceptance-lens-test-tier-clause
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: fixer-start-constraint-misstated
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: usage-record-fan-name-no-criterion
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: d8-restates-listed-constraints
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
---

Round 1 is the first round, so every key is new and open.
The one gating key is `exclusion-union-undefined-across-fan-switch` (F2), a missing decision on D5 against D1's mid-generation fan switch.
The other six are consistency findings between the decision record and its own constraints, the writer stencil or the code it cites.

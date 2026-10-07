---
round: 1
ledger:
  - key: card2-gitrepo-boundary-test-untargeted
    rounds: [1]
    status: open
    class: scope
    severity: BLOCKING
  - key: pushanchored-lock-and-retry-untested
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
  - key: impactsummary-restates-intent-cards-3-4-5
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
---

Round 1 opened three findings and there is no prior ledger.
The BLOCKING one is the plan's card 2 adding a CLI-bound `gitrepo` method without editing the test that pins the bound method set.
The MEDIUM one is a coverage gap on `PushAnchored`'s new lock wait and retry.
The LOW one is three `ImpactSummary` fields that restate `Intent`.

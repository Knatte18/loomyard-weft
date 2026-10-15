---
round: 1
ledger:
  - key: unresolvable-default-guard-untestable
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: card4-engine-names-without-coverage
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: seam-test-file-header-stale
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
---

Round 1 raised three findings, none carried from an earlier round.
The unresolvable-default guard finding spans cards 7, 11 and 12, where test rows use a hand-made engine set whose default is unregistered although the registry build refuses that input.
The `modelspec.EngineNames` finding says card 4 should hand that symbol to card 5, which holds its test.
The seam-test header finding says card 14 should list the whole test file so its stale header is covered.

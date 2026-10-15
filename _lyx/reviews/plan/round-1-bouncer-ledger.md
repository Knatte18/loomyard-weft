---
round: 1
ledger:
  - key: card2-finalize-span-order
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: card13-probe-assertion-unreached
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: card6-parse-stuck-untested
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: card3-gate-span-second-precision
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
---

Round 1 raised four findings and no prior ledger exists.
Card 2's finalize order contradicts its own span-order tests.
Card 13's probe assertion targets an Attach the test never reaches.
Card 6's parse-failure Stuck gains gate fields with no test.
Card 3 stores gate spans to the second, losing sub-second time.
None is gating.

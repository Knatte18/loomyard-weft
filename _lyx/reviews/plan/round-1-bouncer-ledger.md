---
round: 1
ledger:
  - key: relevant-path-wire-assignment-untested
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: io-fault-row-does-not-pin-prefix-change
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: validate-discussion-row-count-mismatch
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 is the first round, so there is no earlier ledger.
F1 maps to `relevant-path-wire-assignment-untested`, F2 to `io-fault-row-does-not-pin-prefix-change` and F3 to `validate-discussion-row-count-mismatch`.
All three are local test-plan corrections whose text the findings already fix.

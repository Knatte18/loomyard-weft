---
round: 2
ledger:
  - key: card6-test-duplicates-engine-tests
    rounds: [1]
    status: resolved
  - key: card8-accumulation-rules-tested-twice
    rounds: [1]
    status: resolved
  - key: card5-plural-tests-for-one-behaviour
    rounds: [1]
    status: resolved
  - key: card14-gated-fork-wait-already-pinned
    rounds: [2]
    status: open
    class: consistency
    severity: MEDIUM
  - key: card8-producer-table-overlaps-existing-tests
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
  - key: card7-fanned-roundtrip-planned-twice
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
  - key: card11-env-fan-test-is-rows-of-existing-tests
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
---

Round 1 raised three test-economy findings on cards 5, 6 and 8.
Round 2's review found no defect on cards 5 and 6, and its card 8 finding is a different overlap at the producer level, so the three round 1 keys are resolved.
Round 2 raised four new test-economy findings: F1 on card 14, F2 on card 8, F3 on card 7 and F4 on card 11.
All are `consistency` class, so none is gating.

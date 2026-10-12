---
round: 1
ledger:
  - key: vocab-name-check-external-test-unit
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: vocab-name-check-rederives-ownedhandles
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: lens-review-prrework-seed-redundant
    rounds: [1]
    status: open
    class: scope
    severity: LOW
  - key: shared-decision-verbatim-overstated
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
---

Round 1 raised four findings.
The blocking one is that card 2's fabric-vocabulary-name check looks up an external test unit (`<dir>_test`) in the owner set unchanged, so it refuses tests the scan admits.
The other three are small: card 2 re-derives `planparser.OwnedHandles`, card 8 seeds a `lens-review.md` in a test that already covers the same path, and the overview's "exactly as the record states" overstates card 9's second-person rewrite of D5.

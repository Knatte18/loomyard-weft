---
round: 2
ledger:
  - key: vocab-name-check-external-test-unit
    rounds: [1]
    status: resolved
  - key: vocab-name-check-rederives-ownedhandles
    rounds: [1]
    status: resolved
  - key: lens-review-prrework-seed-redundant
    rounds: [1]
    status: resolved
  - key: shared-decision-verbatim-overstated
    rounds: [1]
    status: resolved
  - key: cards-name-claim-row-type-not-table
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
---

Round 1 raised four findings, led by the blocking `_test` unit owner lookup in card 2.
Round 2's review did not re-raise any of them, and its focus verdict found card 2's `_test` rule and card 3's row 41 sound, so all four are resolved.
Round 2 raised one new LOW finding: cards 4 and 9 list `stencilClaims` (the row type) where the changed member is `wordingClaims` (the table).

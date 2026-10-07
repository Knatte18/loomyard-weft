---
round: 2
ledger:
  - key: card1-lockdir-error-asserted-twice
    rounds: [1]
    status: resolved
  - key: card4-origin-weft-only-adopt-pinned-twice
    rounds: [1]
    status: resolved
  - key: card6-uptodate-fake-row-redundant
    rounds: [1]
    status: resolved
  - key: card3-failed-adopt-leaves-fetched-branch-unstated
    rounds: [1, 2]
    status: open
    class: design
    severity: LOW
  - key: card4-add-cli-row-asserts-origin-tips-unchanged
    rounds: [2]
    status: open
    class: consistency
    severity: MEDIUM
  - key: card2-checkout-doc-go-missing
    rounds: [2]
    status: open
    class: scope
    severity: LOW
  - key: card4-add-help-text-says-always-forked
    rounds: [2]
    status: open
    class: scope
    severity: LOW
  - key: card6-nil-seams-row-redundant
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
---

Round 1's three duplicated-coverage findings (cards 1, 4 and 6) did not recur in round 2; the focus verdicts found cards 1 and 4 not defective, and card 6's fake row is no longer cited.
Round 1's card 3 gap persists as F2: the bound still omits that a surviving local weft branch reads as a live pair to card 4, now LOW and `design`.
Round 2 added four new open findings: F1 (card 4 CLI row contradicts the adopt rule, MEDIUM, consistency), F3 (card 2 edits no `doc.go`), F4 (the `add` help text in `fabric.go` is stale), and F5 (card 6's nil-seams row duplicates existing Finalize tests).
No open entry is both `design` and MEDIUM or worse.

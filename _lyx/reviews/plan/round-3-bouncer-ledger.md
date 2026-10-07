---
round: 3
ledger:
  - key: webster-callsites-edited-three-times
    rounds: [1, 2, 3]
    status: open
    class: design
    severity: MEDIUM
  - key: profiles-not-declared-open-map
    rounds: [1]
    status: resolved
  - key: tokencount-attribution-ignores-real-transcript-shape
    rounds: [1]
    status: resolved
  - key: card6-repeats-card2-coefficient-error-rows
    rounds: [1, 2]
    status: resolved
  - key: card10-misdescribes-worktree-config-resolution
    rounds: [1]
    status: resolved
  - key: card3-basesha-extraction-untested
    rounds: [2, 3]
    status: open
    class: consistency
    severity: LOW
  - key: card6-testactive-duplicate-error-row
    rounds: [3]
    status: open
    class: design
    severity: LOW
  - key: card7-tail-order-refusal-untested
    rounds: [3]
    status: open
    class: scope
    severity: LOW
---

Round 1 opened five findings; round 2 confirmed three fixed (open-map declaration, tokencount fork attribution, card 10 config resolution).

`webster-callsites-edited-three-times` has persisted through three rounds, narrowing each time.
Round 3 leaves one doubly edited site, the batch-forming line in `internal/webstercli/rebaseline.go`, which card 4 bridges and card 7 replaces.
The proposed fix moves the `RebaselineDeps` shape change into card 4.

`card6-repeats-card2-coefficient-error-rows` is resolved: the negative-coefficient duplication is gone.
Round 3 found a different duplication inside card 6's `TestActive`, tracked as `card6-testactive-duplicate-error-row`.

`card3-basesha-extraction-untested` now has a stated recognition rule and a test.
What remains is the unstated meaning of "earliest" and the fixture wording, so the key stays open at LOW consistency.

`card7-tail-order-refusal-untested` is new in round 3: no test row reaches the rebaseline tail's `CheckBatchOrder` refusal.

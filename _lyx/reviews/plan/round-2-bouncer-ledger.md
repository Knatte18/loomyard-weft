---
round: 2
ledger:
  - key: protocol-read-all-assertion-vacuous
    rounds: [1]
    status: resolved
  - key: read-all-from-combination-untested
    rounds: [1]
    status: resolved
  - key: store-read-doc-comment-unnamed
    rounds: [1]
    status: resolved
  - key: protocol-read-all-assertion-unbuildable-rig
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: read-sets-inbox-sequence-ambiguous
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: reserved-unread-sentences-unnamed
    rounds: [2]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
---

Round 1 raised three findings against card 17; round 2 judged them fixed.
Round 2 raised three new ones against the same card.
The gating one is `protocol-read-all-assertion-unbuildable-rig`: the round-1 fix for the `Request.All` assertion put a second sender with a program claim inside an `asStrand` row, which the identity check refuses on Linux.
The other two are wording corrections to the card's Intent.

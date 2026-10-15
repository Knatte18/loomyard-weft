---
round: 3
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
    status: resolved
  - key: read-sets-inbox-sequence-ambiguous
    rounds: [2, 3]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: reserved-unread-sentences-unnamed
    rounds: [2]
    status: resolved
  - key: protocol-row-old-set-claim-false
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Rounds 1 and 2 raised findings against card 17; the round-1 items and the round-2 BLOCKING rig finding are resolved, as is the round-2 wording finding on reserved and unread sentences.
Round 3 narrows to the `Read_SetsAndInbox` sequence, which still does not pin the oldest-message rule of plain read (scope, MEDIUM), and one false sentence about the old read set in the protocol row (consistency, LOW).
Both fixes are fully specified in the review text.

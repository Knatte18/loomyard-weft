---
round: 4
ledger:
  - key: comment-line-breaks-semantic
    rounds: [1, 2, 3, 4]
    status: open
    class: consistency
    severity: BLOCKING
  - key: remoteleftover-doc-colon-list-split
    rounds: [2]
    status: resolved
  - key: independent-clauses-share-a-line
    rounds: [2, 3, 4]
    status: open
    class: consistency
    severity: BLOCKING
---

Round 1 found changed Go comments hard-wrapped mid-sentence.
Round 2 found the same family at other sites, the `ErrRemoteLeftover` doc splitting a colon-introduced list (closed in round 3), and changed sentences keeping several independent clauses on one line.
Round 3 found breaks inside a clause and independent clauses sharing a line, in non-test and test comments.
Round 4 finds three more sites: `checkout.go:137` (F1) and `remoteleftover.go:2`, `:20` (F2) for clauses sharing a line, and `deps.go:126-127`, `:138` (F3) for a split compound predicate plus a clause join.
All are `PATTERN-comment-line-breaks` findings, gating and BLOCKING.
Each fix clears the named lines and the next review finds new ones, so both open keys are circling.
No code defect was found in any round.

---
round: 3
ledger:
  - key: comment-line-breaks-semantic
    rounds: [1, 2, 3]
    status: open
    class: consistency
    severity: BLOCKING
  - key: remoteleftover-doc-colon-list-split
    rounds: [2]
    status: resolved
  - key: independent-clauses-share-a-line
    rounds: [2, 3]
    status: open
    class: consistency
    severity: BLOCKING
---

Round 1 found changed Go comments hard-wrapped mid-sentence.
Round 2 found the same family at other sites, the `ErrRemoteLeftover` doc splitting a colon-introduced list, and changed sentences keeping several independent clauses on one line.
Round 3 confirms the `ErrRemoteLeftover` doc is correct and closes that key.
It finds breaks inside a clause at `add.go:75-76` and `remoteleftover.go:2-3` (F1), independent clauses sharing a line in non-test comments (F2) and in added test comments (F3).
All are `PATTERN-comment-line-breaks` findings, which the repo makes gating, so each stays BLOCKING.
Each round's fix clears the named sites and the next review finds new ones, so two keys are circling.
No code defect was found in any round.

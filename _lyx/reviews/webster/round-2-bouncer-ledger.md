---
round: 2
ledger:
  - key: comment-line-breaks-semantic
    rounds: [1, 2]
    status: open
    class: consistency
    severity: BLOCKING
  - key: remoteleftover-doc-colon-list-split
    rounds: [2]
    status: open
    class: consistency
    severity: BLOCKING
  - key: independent-clauses-share-a-line
    rounds: [2]
    status: open
    class: consistency
    severity: BLOCKING
---

Round 1 found changed Go comments hard-wrapped mid-sentence in `internal/fabricengine` and `internal/landingshed/finalize.go`.
The round-1 fix moved sentence ends onto their own lines but left wraps inside clauses, so round 2's F1 reports the same family again at `checkout.go`, `mutation.go`, `reconcile.go` and `finalize.go`.
Round 2 also found the `ErrRemoteLeftover` doc splitting a colon-introduced fragment list (F2).
It found changed sentences that keep two independent clauses on one line across the diff (F3).
All three are `consistency` findings under `PATTERN-comment-line-breaks`, rated BLOCKING.
No code defect was found.

---
round: 2
ledger:
  - key: d6-fork-point-and-delete-rules
    rounds: [1]
    status: resolved
  - key: d4-empty-overview-unguarded-by-d2
    rounds: [1, 2]
    status: open
  - key: d9-move-list-deferred-to-board
    rounds: [1]
    status: resolved
  - key: d14-dangling-open-risks-reference
    rounds: [1]
    status: resolved
  - key: d2-forked-pair-failure-unbounded
    rounds: [1]
    status: resolved
  - key: d13-rejected-alternative-in-decision
    rounds: [1]
    status: resolved
  - key: d13-portals-anchor-lifecycle
    rounds: [1, 2]
    status: open
  - key: acceptance-gaps-d7-d11
    rounds: [1]
    status: resolved
  - key: d6-stuck-rerun-unspecified
    rounds: [2]
    status: open
---

Round 1 raised eight findings; round 2 raised three, none above LOW.
Round 2's fresh review confirmed the D6 fork point and delete rules, the D9 move list, the D14 reference and the D2 failure bound as settled, and found no rejected alternative left in a Decision.
Still open, all LOW or NIT: an overview with no entry bullet still passes D2's checker (F1), the D6 re-run after a Stuck carry is unspecified (F2), and the D13 anchor-directory lifecycle is half stated (F3).

---
round: 1
ledger:
  - key: ring-row-posts-two-messages-in-one-settle
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
---

Round 1 raised one LOW scope finding (F1): `ARingAnsweredByAClaimIsNotFollowedByTheBell` posts two messages inside one settle trigger.
The fix is local: post before the start, then settle on the start.

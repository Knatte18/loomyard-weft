---
round: 2
ledger:
  - key: relaunch-wording-no-third-file-carrier
    rounds: [1]
    status: resolved
  - key: self-check-passes-skipped-relevant-file
    rounds: [1]
    status: resolved
  - key: d2-never-search-further-subject
    rounds: [1]
    status: resolved
  - key: path-rule-bound-overclaims
    rounds: [1, 2]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: duplicate-head-start-descriptions
    rounds: [1]
    status: resolved
  - key: d1-plan-host-relaunch-value
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
---

Round 1's relaunch, self-check, D2-subject and head-start findings are not reraised in round 2, so they are resolved.
The path-rule finding persists as F2: the `..` separator set is still open and the Bound overclaims, now LOW and design class.
F1 is new in round 2: D1's Plan-host clause, added by the round-1 fix, rests on a false premise.
Its fix text is given in the review.

---
round: 2
ledger:
  - key: d2-rearm-bound-misreads-loom-start-resume
    rounds: [1]
    status: resolved
  - key: d4-has-no-own-acceptance-criterion
    rounds: [1]
    status: resolved
  - key: acceptance-row-overview-replace-contradicts-d6
    rounds: [1]
    status: resolved
  - key: d4-bound-incomplete-after-d5-bound
    rounds: [1]
    status: resolved
  - key: d3-miscounts-focus-users
    rounds: [1]
    status: resolved
  - key: d7-degraded-click-undecided
    rounds: [1]
    status: resolved
  - key: loom-resume-verb-not-in-or-out-of-d2-rearm
    rounds: [2]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: ending-window-row-disagrees-with-d6-replacestrand
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1's six findings are settled by the round 2 review, which checks each against the revised record and raises none of them again.
Round 2 raised two new keys.
The `lyx loom resume` verb is left out of D2's bound while D2's rationale implies it re-arms follow; the fix adds a decision sentence and an acceptance row.
The ending-window acceptance row ignores that D6 keeps the overview replacement's move to the new overview window; the fix is a rewording.

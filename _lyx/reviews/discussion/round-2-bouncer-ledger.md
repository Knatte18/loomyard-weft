---
round: 2
ledger:
  - key: d2-pattern-entry-overview-headroom
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d4-route-does-not-carry-a-drop
    rounds: [1]
    status: resolved
  - key: d6-plan-review-acceptance-unbounded
    rounds: [1]
    status: resolved
  - key: edit-wider-than-board-lock
    rounds: [1, 2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d5-darn-no-route-for-own-entry-step
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
---

Round 1's BLOCKING headroom finding is gone, but D2 is still open: the review now says a workflow rule does not belong in PATTERN, and the fix moves the rule into D1 and `CLAUDE.md`.
The D4 and D6 findings are resolved; the review's focus verdicts found no defect there.
The darn-path finding recurs and rose from LOW to MEDIUM: D5 lets the darn writer drop a step before the record carries the drop.
The wording finding on "edit" persists as a local correction, now naming D1 and D3 to D6.

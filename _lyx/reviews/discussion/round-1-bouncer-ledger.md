---
round: 1
ledger:
  - key: d2-pattern-entry-overview-headroom
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: d4-route-does-not-carry-a-drop
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d6-plan-review-acceptance-unbounded
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: edit-wider-than-board-lock
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d5-darn-no-route-for-own-entry-step
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
---

Round 1 is the first round; no earlier ledger exists.
The BLOCKING finding is that D2's new PATTERN entry exceeds the overview headroom and the record names no cut.
D4, D6 and D5 share one theme: a dropped own-entry step has no recorded route or bound in the Plan and darn paths.
The wording finding on "edit" is a local correction.

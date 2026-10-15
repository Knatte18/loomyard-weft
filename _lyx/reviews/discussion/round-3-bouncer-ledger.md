---
round: 3
ledger:
  - key: d8-carrier-task-summary-false-premise
    rounds: [1, 2, 3]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: d8-writer-orch-edge-unbounded
    rounds: [3]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d8-q9-state-disagrees-with-decision
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: pattern-read-set-three-wordings
    rounds: [1]
    status: resolved
  - key: empty-pattern-directive-criterion-unmeetable
    rounds: [1]
    status: resolved
  - key: d3-replaced-span-ambiguous
    rounds: [1]
    status: resolved
  - key: d1-read-ban-bound-unstated
    rounds: [1]
    status: resolved
  - key: d3-fragment-claims-rules-current
    rounds: [2]
    status: resolved
  - key: goal-pattern-read-set-wider-than-d1
    rounds: [2]
    status: resolved
  - key: d3-rest-unchanged-wrong-for-plain-fragment
    rounds: [2]
    status: resolved
---

The D8 carrier has been open since round 1.
Round 1 named a task-summary carrier on a false premise, round 2 found the board-note carrier was never handed to the writer, and round 3 finds the writer's message to `ly:orch` has no scheduled sender.
It is now a `decision`-class finding, so it no longer gates.

Round 3 adds the unbounded writer-to-orch routing edge (F2) and the D8/Q9 state disagreement (F3).
The three round-2 wording findings were fixed and no longer appear.

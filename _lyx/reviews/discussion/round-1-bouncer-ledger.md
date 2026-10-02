---
round: 1
ledger:
  - key: d1-shape-mismatch-unbounded
    rounds: [1]
    status: resolved
  - key: d1-list-element-fill-unspecified
    rounds: [1]
    status: resolved
  - key: notes-reconcile-reuse-hazards
    rounds: [1]
    status: resolved
  - key: goal-vs-one-time-spawn
    rounds: [1]
    status: resolved
  - key: d2-liveness-test-unnamed
    rounds: [1]
    status: resolved
  - key: d4-tracked-tasks-json-branch
    rounds: [1]
    status: resolved
  - key: d6-board-exemption-bound
    rounds: [1]
    status: resolved
  - key: d8-no-acceptance-criterion
    rounds: [1]
    status: resolved
---

Round 1 raised eight findings against the discussion pair, all of which the current `decision-record.md` already resolves.
The two MEDIUM ones were D1's fill silently overwriting a present value of the wrong shape, now refused by name, and the Goal promising no manual step while pre-landing worktrees need one `lyx ide spawn`, now stated in the Goal, assumptions and Open risks.
The LOW ones tightened D1's list-element and extra-key behaviour, the Notes' warning about reusing `yamlengine.Reconcile`, D2's liveness definition and platform parity, D4's tracked-`tasks.json` branch, D6's weft-side exemption, and D8's missing acceptance criterion.
No finding stays open.

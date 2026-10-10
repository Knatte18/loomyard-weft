---
round: 3
ledger:
  - key: d1-timeout-unstartable-no-failure-record
    rounds: [1]
    status: resolved
  - key: d3-crashed-driver-leftover-still-refused
    rounds: [1]
    status: resolved
  - key: d6-standalone-run-slug-free-premise
    rounds: [1, 2]
    status: resolved
  - key: d6-hardcoded-hub-shortname
    rounds: [1]
    status: resolved
  - key: d5-conditional-fail-branch-undesigned
    rounds: [1]
    status: resolved
  - key: d1-bound-overstates-reverify
    rounds: [1]
    status: resolved
  - key: d2-bound-misses-hand-started-merge-in
    rounds: [1]
    status: resolved
  - key: acceptance-pins-already-built-behavior
    rounds: [1]
    status: resolved
  - key: constraints-carry-no-requirement
    rounds: [1]
    status: resolved
  - key: open-risks-restate-bounds
    rounds: [1]
    status: resolved
  - key: resume-dead-driver-criterion-contradicts-d3
    rounds: [2]
    status: resolved
  - key: d1-goto-named-when-failure-record-unwritten
    rounds: [2]
    status: resolved
  - key: d3-conflict-session-outlives-driver
    rounds: [2, 3]
    status: open
    class: decision
    severity: MEDIUM
  - key: d3-no-live-driver-way-forward-misstated
    rounds: [3]
    status: open
    class: design
    severity: LOW
  - key: d6-unresolvable-parent-error-path
    rounds: [3]
    status: open
    class: decision
    severity: LOW
---

Round 1 raised ten findings, round 2 four, round 3 three.
`d6-standalone-run-slug-free-premise`, the only gating key recurring across rounds, is not re-raised in round 3 and its focus verdict calls the premise not a defect, so it is resolved.
`resume-dead-driver-criterion-contradicts-d3` and `d1-goto-named-when-failure-record-unwritten` are likewise not re-raised; the focus verdicts confirm the acceptance wording and D1's record-write case.
`d3-conflict-session-outlives-driver` continues as F1: the stop is now stated, and the open path is a failed stop (MEDIUM, class decision).
F2 is a new key, D3 names `lyx loom start` for a no-live-driver refusal that gives batten's revive for a dead strand.
F3 is a new key, D6 leaves the `hubgeom.ResolveParent` error path undecided.

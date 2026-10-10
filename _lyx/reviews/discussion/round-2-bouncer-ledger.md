---
round: 2
ledger:
  - key: d1-timeout-unstartable-no-failure-record
    rounds: [1]
    status: resolved
  - key: d3-crashed-driver-leftover-still-refused
    rounds: [1]
    status: resolved
  - key: d6-standalone-run-slug-free-premise
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
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
    status: open
    class: consistency
    severity: MEDIUM
  - key: d1-goto-named-when-failure-record-unwritten
    rounds: [2]
    status: open
    class: design
    severity: LOW
  - key: d3-conflict-session-outlives-driver
    rounds: [2]
    status: open
    class: design
    severity: LOW
---

Round 1 raised ten findings; round 2 raised four.
Nine of the round-1 keys are resolved: the round-2 review does not re-raise them, and its focus verdicts confirm the D1 timeout route, D2's bound and the D6 literal shortname.
`d6-standalone-run-slug-free-premise` stays open: F2 shows the slug-free premise is still false for a pair created from a task worktree, where `hubgeom` records a slug-bearing parent.
The other three round-2 findings are new keys: the acceptance criterion claims `resume` proceeds for a dead driver (F1), D1 names the goto when the failure record write failed (F3), and D3 is silent on a conflict session that outlived its driver (F4).

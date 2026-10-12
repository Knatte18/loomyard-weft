---
round: 3
ledger:
  - key: replay-vs-harvest-auto-accept-outcome
    rounds: [1, 2, 3]
    status: open
    class: consistency
    severity: MEDIUM
  - key: soft-hard-cap-measure-undefined
    rounds: [1]
    status: resolved
  - key: step4-returning-key-test-underdefined
    rounds: [1, 2]
    status: resolved
  - key: mechanical-mistag-bound-missing
    rounds: [1, 2, 3]
    status: open
    class: design
    severity: MEDIUM
  - key: budget-escalation-at-hard-cap-offers-continue
    rounds: [1]
    status: resolved
  - key: hard-cap-escalation-over-recorded-continue-unanswerable
    rounds: [1, 2]
    status: resolved
  - key: orch-stencil-change-has-no-decision
    rounds: [1]
    status: resolved
  - key: rule-continue-cause-unstated
    rounds: [1]
    status: resolved
  - key: hard-cap-bypassed-after-goto-count-reset
    rounds: [2]
    status: resolved
  - key: hard-cap-way-forward-misses-run-settings-layer
    rounds: [2, 3]
    status: open
    class: design
    severity: BLOCKING
  - key: hard-cap-no-round-after-h-overstated
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
---

Round 3 closed the goto bypass, the escalation over a recorded continue, and the step 4 returning-key premise.
The hard-cap way forward is the persistent problem: round 2 found it missed the `run_settings` layer, and round 3 finds the replacement route still never reaches the running task worktree, now BLOCKING.
The mechanical-mistag bound has been open since round 1 and is now wrong about PR-Gate review and Merge-Review's downstream segments.
The acceptance criteria still lack the auto-accept qualifier (MEDIUM, up from LOW).
A new LOW finding says D3 overstates that no round after round H runs.

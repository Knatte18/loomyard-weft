---
round: 1
ledger:
  - key: gate-rearrives-under-running-fixer
    rounds: [1]
    status: resolved
  - key: d5-non-merge-rule-refuses-parent-merge
    rounds: [1]
    status: resolved
  - key: d5-path-check-cannot-fire
    rounds: [1]
    status: resolved
  - key: d7-record-writer-unbounded
    rounds: [1]
    status: resolved
  - key: d8-stuck-edge-unbounded
    rounds: [1]
    status: resolved
  - key: d1-card-gate-unenforced-unstated
    rounds: [1]
    status: resolved
  - key: d1-contradicts-spec-tier2
    rounds: [1]
    status: resolved
  - key: d3-absent-outcome-pass-unbounded
    rounds: [1]
    status: resolved
  - key: acceptance-criteria-miss-decisions
    rounds: [1]
    status: resolved
  - key: d4-incident-narrative
    rounds: [1]
    status: resolved
---

Round 1 reviewed the first draft and found ten issues.
The BLOCKING one was a false Constraints claim that the webster gate never verifies under a running fork: Merriam's output files stay on disk after the first gated Done, so the fixer-spawning turn end would have re-arrived at the gate.
D3 now makes the `EventWaiting` deferral a requirement with a `master_timeout_min` bound.
The MEDIUM findings tightened D5's commit check (accept clean parent merges, rest plan and `_lyx` writes on `CheckFork`), D7's record-writer bound, and D8's Stuck-edge bound.
The LOW and NIT findings added D1's bound and Tier 2 note, D3 step 1's absent-outcome reasoning and the missing acceptance criteria, and cut incident narrative from D4.
The fixer resolved all ten in `decision-record.md`, and the judge approved round 1.

---
round: 3
ledger:
  - key: boot-signal-test-not-discriminating
    rounds: [1]
    status: resolved
  - key: orch-hold-memory-not-forgotten-on-pass
    rounds: [1]
    status: resolved
  - key: orch-hold-stuck-survives-restart
    rounds: [1]
    status: resolved
  - key: idle-cpu-report-unmeasured-stat-tick-claim
    rounds: [1]
    status: resolved
  - key: shuttle-wait-leftover-default-constant
    rounds: [1]
    status: resolved
  - key: pattern-md-over-size-cap-after-merge
    rounds: [3]
    status: open
    class: design
    severity: BLOCKING
---

Round 1 raised five findings, all fixed and confirmed resolved in round 2.
Round 3 raised one BLOCKING finding: the merge-in of `main` left `PATTERN.md` 192 bytes over the 16384 cap, failing `TestOverview_RealFilePassesCheck`.
The branch's `PATTERN-daemon-wakeups` and `fswatch` additions are minimal; the cut has to come from tightening other entries.
Round 3's review found no other defect in the branch's range.

---
round: 2
ledger:
  - key: publish-comments-stale-merge-in
    rounds: [1]
    status: resolved
  - key: lyxtemp-concurrent-sweep-lock-probe-race
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
---

Round 1 raised one NIT about stale merge-in wording in two `Publish` comments; round 2 did not raise it again, so it is resolved.
Round 2 raised one BLOCKING design finding: during concurrent sweeps in `internal/lyxtemp`, one sweeper's lock probe recreates the lock file inside a run directory the other is removing, so the removal fails with "directory not empty" and `TestSweep/two_concurrent_sweeps_both_succeed` fails.

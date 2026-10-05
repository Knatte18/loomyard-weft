---
round: 2
ledger:
  - key: compact-survival-premise-unrecorded
    rounds: [1]
    status: resolved
  - key: notice-queue-lifecycle-unbounded
    rounds: [1, 2]
    status: open
  - key: notice-content-prefix-unbounded
    rounds: [1]
    status: resolved
  - key: stale-permission-mode-config-failure-mode
    rounds: [1, 2]
    status: open
  - key: requested-cycle-skips-reread-unbounded
    rounds: [1]
    status: resolved
  - key: registry-process-start-assumption-unverified
    rounds: [1]
    status: resolved
  - key: stuck-reason-no-release-condition
    rounds: [1]
    status: resolved
  - key: quiet-condition-overlap-missing-criteria
    rounds: [1]
    status: resolved
  - key: process-narrative-and-landing-action-in-record
    rounds: [1]
    status: resolved
  - key: d7-defer-guard-removal-unbounded
    rounds: [2]
    status: open
---

Round 2 raised three findings.
F1 (BLOCKING, design) maps to `stale-permission-mode-config-failure-mode`: D10 names the wrong loader for the orch config.
F2 (MEDIUM, design) is new: `d7-defer-guard-removal-unbounded`.
The round-1 premise is now recorded, but D7 itself states no bound.
F3 (LOW, decision) maps to `notice-queue-lifecycle-unbounded`: the cap and drop rule are defined, the storage form is not.
Round 1's other keys were not re-raised by the fresh review and are marked resolved.

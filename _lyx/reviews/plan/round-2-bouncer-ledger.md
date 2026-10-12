---
round: 2
ledger:
  - key: lock-wait-not-watchdog-activity
    rounds: [1]
    status: resolved
  - key: fastforward-pair-stale-read-bound
    rounds: [1]
    status: resolved
  - key: parentlock-wait-escapes-daemon-wakeups-guard
    rounds: [1]
    status: resolved
  - key: card-17-bundles-rubric-stencil
    rounds: [1]
    status: resolved
  - key: producer-tests-duplicate-lower-layer
    rounds: [1]
    status: resolved
  - key: no-test-reaches-merge-in-progress-mapping
    rounds: [1]
    status: resolved
  - key: force-release-unbounded-in-card-1
    rounds: [1]
    status: resolved
  - key: merge-review-off-reader-wrong-card
    rounds: [1]
    status: resolved
  - key: fastforward-pair-mutation-record-shape
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
  - key: status-spec-omits-lock-wait
    rounds: [2]
    status: open
    class: scope
    severity: MEDIUM
---

Round 1 raised eight findings, led by the lock-wait watchdog gap and the fast-forward stale-read bound.
Round 2 no longer raises any of them, and its focus verdicts confirm the watchdog, fast-forward ordering, daemon-wakeups guard and refusal-reach coverage.
Round 2 raised two new findings: `FastForwardPair` lacks a `MutationRecord` result and `SyncOptions` input (BLOCKING, design), and card 22 leaves `loom-status-spec.md` unedited for the fifth wait (MEDIUM, scope).

---
round: 1
ledger:
  - key: lock-wait-not-watchdog-activity
    rounds: [1]
    status: open
    class: scope
    severity: BLOCKING
  - key: fastforward-pair-stale-read-bound
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: parentlock-wait-escapes-daemon-wakeups-guard
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
  - key: card-17-bundles-rubric-stencil
    rounds: [1]
    status: open
    class: design
    severity: LOW
  - key: producer-tests-duplicate-lower-layer
    rounds: [1]
    status: open
    class: design
    severity: LOW
  - key: no-test-reaches-merge-in-progress-mapping
    rounds: [1]
    status: open
    class: scope
    severity: LOW
  - key: force-release-unbounded-in-card-1
    rounds: [1]
    status: open
    class: design
    severity: LOW
  - key: merge-review-off-reader-wrong-card
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
---

Round 1 is the first round, so no earlier ledger exists.
The gating items are the watchdog gap (F1) and the fast-forward bound that depends on a read made before `Pull`'s own fetch (F2).
The remaining findings are test, pattern-guard and cross-reference gaps.

---
round: 3
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
    status: resolved
  - key: status-spec-omits-lock-wait
    rounds: [2]
    status: resolved
  - key: merged-tip-uses-prefetch-upstream
    rounds: [3]
    status: open
    class: design
    severity: MEDIUM
  - key: fastforward-pair-weft-step-unstated
    rounds: [3]
    status: open
    class: design
    severity: MEDIUM
  - key: status-spec-lock-wait-writer-wording
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
---

Round 1 raised eight findings, led by the lock-wait watchdog gap and the fast-forward stale-read bound; none recurred.
Round 2 raised the missing `FastForwardPair` result record and the unedited status spec; round 3 confirms the signature and result sound and the spec edit made.
Round 3 raised three new findings: Merge-In records the pre-fetch upstream tip instead of `NewHead` (card 12), the `FastForwardPair` weft step leaves upstream, failure and recording semantics unstated (card 7), and the status-spec text names Merge-In as the only lock-wait writer (card 22).

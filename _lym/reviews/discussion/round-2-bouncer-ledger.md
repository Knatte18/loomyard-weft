---
round: 2
ledger:
  - key: d4-way-forward-per-site
    rounds: [1]
    status: resolved
  - key: d6-profiles-collision-drops-operator-edit
    rounds: [1]
    status: resolved
  - key: d7-retry-only-next-build
    rounds: [1]
    status: resolved
  - key: run-settings-set-one-token-name
    rounds: [1]
    status: resolved
  - key: d5-prime-seed-not-moved
    rounds: [1]
    status: resolved
  - key: mixed-build-risk-overstates-refusal
    rounds: [1]
    status: resolved
  - key: d6-criterion-byte-equality
    rounds: [1]
    status: resolved
  - key: d5-walk-not-before-inflight-load
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
---

Round 1's seven findings are all resolved by the round 2 review's prior-finding answers.
Round 2 raised one new finding, F8: `hubreconcile.Ensure` runs only from start verbs, so after the rebuild an in-flight run's next non-start verb loads a still-flat file and D4 refuses it.
Its fix is a Decision in D5 that either runs the stamp-gated reconcile before every loom or tapestry config load, or has the loader read old keys at their new paths in memory while the stamp is stale, plus a matching bound and acceptance criterion.

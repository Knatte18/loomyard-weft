---
round: 3
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
    status: resolved
---

Round 1's seven findings were resolved in round 2.
Round 2's F8 (`d5-walk-not-before-inflight-load`) is resolved in round 3: D5 now runs `hubreconcile.EnsureOrWarn` before every non-`lym config` loom or tapestry config load, with matching bounds and an acceptance criterion.
Round 3 raised no new finding.

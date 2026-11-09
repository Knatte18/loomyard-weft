---
round: 2
ledger:
  - key: syncoptions-notifier-fallback-unpinned
    rounds: [1]
    status: resolved
  - key: newtopology-gc-threshold-unpinned
    rounds: [1]
    status: resolved
---

Round 1 raised two test-coverage gaps in `internal/fabricengine`: the unpinned `SyncOptions.notifier` precedence and fallback, and the unpinned default `gc.auto` threshold in `NewTopology`.
Round 2 confirmed both are now pinned by exact-count tests, and raised no new findings.

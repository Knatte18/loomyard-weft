---
round: 1
ledger:
  - key: syncoptions-notifier-fallback-unpinned
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: newtopology-gc-threshold-unpinned
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised two test-coverage gaps in `internal/fabricengine`.
The first is the unpinned `SyncOptions.notifier` precedence and installed-notifier fallback (MEDIUM, gating).
The second is the unpinned default `gc.auto` threshold in `NewTopology` (LOW).
Both fixes are fully specified by the finding text, so both are mechanical.

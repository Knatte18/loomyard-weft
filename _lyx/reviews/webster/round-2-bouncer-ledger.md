---
round: 2
ledger:
  - key: comment-hard-wrap-mid-sentence
    rounds: [1, 2]
    status: open
    class: consistency
    severity: LOW
  - key: new-tests-missing-t-parallel
    rounds: [1]
    status: resolved
  - key: commitwritten-uncovered-behaviors
    rounds: [1]
    status: resolved
  - key: configengine-doc-rewrap-note-stale
    rounds: [1]
    status: resolved
  - key: reconcile-help-omits-hubwide-repair
    rounds: [1]
    status: resolved
  - key: configengine-note-lost-dashes
    rounds: [2]
    status: open
    class: consistency
    severity: NIT
  - key: reconcile-hub-config-detail-untested
    rounds: [2]
    status: open
    class: scope
    severity: LOW
---

Round 1's parallel-test, `Bolt.CommitWritten` coverage, configengine note and reconcile help findings are resolved: round 2 raises none of them again.
The comment line-break key reappears in round 2 as F1, at LOW, on two comments the fix commits rewrote or extended.
Two new findings are open: F2, a NIT where the configengine note lost its dashes, and F3, a LOW coverage gap for reconcile's non-fatal `hub_config_detail` path.
No open entry is `design` class, and none is BLOCKING.

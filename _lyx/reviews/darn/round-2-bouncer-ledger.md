---
round: 2
ledger:
  - key: fallback-misses-permission-denied-journal
    rounds: [1]
    status: resolved
  - key: pause-record-wiring-untested
    rounds: [1]
    status: resolved
  - key: stale-sleepjournal-file-header
    rounds: [1]
    status: resolved
---

Round 1 raised three findings: the gating `fallback-misses-permission-denied-journal`, the test-strength `pause-record-wiring-untested`, and the header nit `stale-sleepjournal-file-header`.
Round 2 reviewed the fixes fresh and found nothing, so all three are resolved.

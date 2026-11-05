---
round: 1
ledger:
  - key: readiness-fallback-sessionidle-test-gap
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
---

Round 1 raised F1: the `readinessState` fallback is pinned only through the start path, with no rows for `Runner.SessionIdle` and a stale unmatched turn start before the prompt offset.
The fix names the three rows to add to the existing table, so it is a local correction.

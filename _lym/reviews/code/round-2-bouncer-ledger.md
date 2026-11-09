---
round: 2
ledger:
  - key: start-cost-bound-measured-time-vs-pattern
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: stale-testtiming-keep-marker
    rounds: [1]
    status: resolved
---

Round 1 raised two findings on `internal/battencli/lifecycle_integration_test.go`.
The stale keep marker is resolved: round 2 found the marker and the constant's comment agree.
The measured-time bound recurs in round 2: a 2 s `startCostBound` is within reach of a loaded machine, so it does not meet the exception's "far above load" condition.
The round 2 fix text asks for a 30 s bound and a comment that follows it.

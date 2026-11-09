---
round: 1
ledger:
  - key: start-cost-bound-measured-time-vs-pattern
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: stale-testtiming-keep-marker
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised two findings, both on `internal/battencli/lifecycle_integration_test.go`.
The first is that `startCostBound` still compares measured wall time to a fixed bound, against the rule the diff adds to `PATTERN-test-speed.md`.
The second is that the `testtiming:keep` marker still cites the old 100 ms bound.

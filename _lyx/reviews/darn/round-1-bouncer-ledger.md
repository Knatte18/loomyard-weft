---
round: 1
ledger:
  - key: dead-loop-stale-child-named-in-flight
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: killed-tree-wording-untested
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: early-exit-hook-arguments-untested
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: weak-trace-id-assertion
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised four findings.
F1 is gating: the dead-loop cause and `trace_file` name the pid record's last child as in flight even after the status moved on.
F2 to F4 are test-coverage gaps in `internal/shedverbs`.

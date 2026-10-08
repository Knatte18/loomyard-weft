---
round: 1
ledger:
  - key: low-speed-abort-not-transport-failure
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: gc-auto-misses-auto-maintenance
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: d1-lock-discipline-open
    rounds: [1]
    status: open
    class: decision
    severity: LOW
  - key: d3-timeout-consequence-unbounded
    rounds: [1]
    status: open
    class: design
    severity: LOW
  - key: acceptance-criteria-gaps-d2-d3
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: gc-pattern-entry-lacks-chokepoint-carveout
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: q6-447-tension-wording-mismatch
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: d2-rationale-snapshot-figures
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
---

Round 1 raised eight findings, none carried from an earlier round.
Two are gating: the low-speed abort that D3 relies on is not matched by `IsTransportFailure`, and `gc.auto=0` in D2 leaves `maintenance.auto` active.
The rest are lock discipline in D1, D3's timeout bound, missing acceptance criteria, a PATTERN carve-out, a Q6 wording mismatch and a snapshot-figure nit.

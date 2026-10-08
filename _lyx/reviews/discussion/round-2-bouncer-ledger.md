---
round: 2
ledger:
  - key: low-speed-abort-not-transport-failure
    rounds: [1]
    status: resolved
  - key: gc-auto-misses-auto-maintenance
    rounds: [1]
    status: resolved
  - key: d1-lock-discipline-open
    rounds: [1]
    status: resolved
  - key: d3-timeout-consequence-unbounded
    rounds: [1, 2]
    status: open
    class: design
    severity: LOW
  - key: acceptance-criteria-gaps-d2-d3
    rounds: [1]
    status: resolved
  - key: gc-pattern-entry-lacks-chokepoint-carveout
    rounds: [1]
    status: resolved
  - key: q6-447-tension-wording-mismatch
    rounds: [1]
    status: resolved
  - key: d2-rationale-snapshot-figures
    rounds: [1]
    status: resolved
  - key: d1-split-lock-retry-race
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
  - key: gc-removes-more-than-unreachable
    rounds: [2]
    status: open
    class: design
    severity: LOW
  - key: d6-gc-ownership-overstated
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
  - key: d3-not-every-git-call-through-runcore
    rounds: [2]
    status: open
    class: design
    severity: LOW
---

Round 1 raised eight findings; round 2's focus verdicts settle the low-speed abort, `maintenance.auto` and D1 lock-order questions, and the other round-1 keys are not raised again.
Round 2 raised five findings.
The gating one, F1, is the race the D1 lock split opens, filed as `d1-split-lock-retry-race`.
F5 reopens the D3 bound as a per-command rather than per-step matter.
F2 to F4 are new LOW keys on gc deletion scope, D6's ownership wording and `tools/deploy`'s git calls.

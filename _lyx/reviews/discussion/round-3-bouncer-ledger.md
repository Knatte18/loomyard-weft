---
round: 3
ledger:
  - key: sandbox-list-vs-unchanged-bytes
    rounds: [1, 2]
    status: resolved
  - key: d7-forkaudit-dependency-no-fallback
    rounds: [1, 2, 3]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d3-bound-omits-trace-write
    rounds: [1, 2, 3]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: start-here-confound-in-open-risks
    rounds: [1]
    status: resolved
  - key: d6-refusal-info-line-undefined
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 3 left only LOW consistency findings.
The D7 blocker shrank to F1: the record pins `tools/forkaudit` names the audit entry does not set, and the fix replaces them with capability wording.
The D3 bound still misses the Warn's stderr output (F2).
F3 is new: D6 promises an Info line for every call, but a refused call has no counts.
The shared-body sandbox list drew no finding and is resolved.

---
round: 1
ledger:
  - key: loom-help-removed-review-verb
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: shedrun-alignment-helpers-outlive-agent
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: reportengine-archived-decoders-untested
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
---

Round 1 raised three scope findings (F1, F2, F3) and no gating one.
F1 is the removed `lyx loom review approve` example in the loom help; F2 is the leftover alignment directory helpers and staging; F3 is the missing direct tests for the archived-record decoders.

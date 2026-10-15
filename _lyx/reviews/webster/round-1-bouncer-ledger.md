---
round: 1
ledger:
  - key: frictionengine-reflection-spec-stale-comment
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: mergeresolve-conflict-spec-stale-comment
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised two LOW consistency findings: the doc comments of `buildReflectionSpec` (F1) and `buildConflictSpec` (F2) still describe the retired field copy instead of `shuttleengine.NewModelChoice`.
Each fix is a one-sentence comment edit.
No gating finding was raised.

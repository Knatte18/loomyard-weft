---
round: 1
ledger:
  - key: rubric-drops-actually-ran
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: declared-bullet-unspecified-action
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised two non-gating consistency findings.
F1 (`rubric-drops-actually-ran`): the per-card mechanical-check bullet lost "actually ran" for every card type.
F2 (`declared-bullet-unspecified-action`): the `declaredBullet` fixture used an action the decision-record spec does not define.
Both fixes are local wording corrections.

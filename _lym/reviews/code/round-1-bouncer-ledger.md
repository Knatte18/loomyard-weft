---
round: 1
ledger:
  - key: handover-doc-overstates-batten-pickup
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised F1: the `handOver` doc and the change description say a failed hand-over still reaches batten, but batten's binary re-step skips a tree with uncommitted paths.
The review states the exact replacement text for both statements, so the fix is a local wording correction.
No earlier ledger exists.

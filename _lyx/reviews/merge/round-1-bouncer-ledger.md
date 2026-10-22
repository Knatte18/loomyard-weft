---
round: 1
ledger:
  - key: pattern-md-grows-past-headroom
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: pause-vs-owner-refusal-order-unpinned
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised two findings.
The merged `PATTERN.md` lost most of both sides' shortening and sits past the headroom line (F1, gating).
No test pins the pause refusal ahead of the owner refusal in `lyx loom start` and `lyx loom resume` (F2).
Both fixes are local corrections the finding text already fixes.

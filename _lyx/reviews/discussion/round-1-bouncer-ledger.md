---
round: 1
ledger:
  - key: text-equality-criterion-compares-rendered-with-raw-bodies
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d4-skipped-pack-write-let-through-outside-decision
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised two LOW findings, both with exact replacement text in the review.
F1 (consistency) is the acceptance criterion comparing rendered text with raw marker-bearing fragment bodies.
F2 (design) is D4's stale `<batch>-pack.md` let-through stated only in Scope Out.
Neither gates; the loop is converged.

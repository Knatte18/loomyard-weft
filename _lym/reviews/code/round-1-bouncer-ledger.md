---
round: 1
ledger:
  - key: design-class-faults-every-chain
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: concurrency-judgement-measures-chain-length-only
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised two design findings against the rubric stencil's concurrency wording.
F1 (MEDIUM) maps to `design-class-faults-every-chain`; F2 (LOW) maps to `concurrency-judgement-measures-chain-length-only`.
Both fixes are local text replacements the review spells out.

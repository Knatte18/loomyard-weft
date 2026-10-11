---
round: 1
ledger:
  - key: haiku-judge-note-stale-judge-fields
    rounds: [1]
    status: open
    class: scope
    severity: LOW
  - key: judgment-refusal-unreachable-ledger-reread
    rounds: [1]
    status: open
    class: design
    severity: NIT
---

Round 1 is the first round; there was no previous ledger.
F1 maps to `haiku-judge-note-stale-judge-fields` and F2 to `judgment-refusal-unreachable-ledger-reread`.
Both are non-gating, so the loop converged.

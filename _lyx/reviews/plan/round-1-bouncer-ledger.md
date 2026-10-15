---
round: 1
ledger:
  - key: stencil-caller-rule-offers-test-file
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: rework-parity-fixture-no-impact-summary
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: rubric-sentence-quoted-without-too
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised three findings, none gating.
The stencil's caller rule still offers a test file's path (F1), the rework parity fixture lacks an `ImpactSummary` (F2), and card 3 misquotes a rubric sentence (F3).
Each fix is a local text correction the finding already spells out.

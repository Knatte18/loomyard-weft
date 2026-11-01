---
round: 1
ledger:
  - key: former-default-strip-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: sandbox-doc-names-cautious-default
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: wide-vs-cautious-budget-only-unpinned
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised three findings.
The former-default strip of batcher `active: "cautious"` lacks a test row (F1, MEDIUM, gating).
The sandbox suite doc still names `cautious` as the default (F2, LOW).
Nothing pins that `wide` differs from `cautious` only by budget (F3, LOW).
All three are local corrections whose result the finding text fixes.

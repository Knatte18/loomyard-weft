---
round: 1
ledger:
  - key: ask-verb-posts-second-ask-past-satisfied
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: stencil-await-rule-asks-again-past-satisfied
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: stencil-it-lost-antecedent
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: prreviewawaiting-doc-omits-satisfied-precedence
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised four findings, F1 to F4, in that order of the keys above.
F1 is the only gating one: the ask verb lacks a refusal where a satisfied answer stands at the head.
F2 to F4 are stencil and doc consistency fixes.
F1 is marked `design-wrong` because it adds a new refusal to the verb's contract, which a fresh review should see.

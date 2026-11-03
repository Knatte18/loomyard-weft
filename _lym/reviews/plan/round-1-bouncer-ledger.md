---
round: 1
ledger:
  - key: card5-every-engine-build-test-unlisted
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: recipe-spec-stale-for-block-rows
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: arras-motif-interrupt-policy-missing
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: card8-unreachable-profile-recipe-fallback
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: docs-list-recipes-without-arras
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: block-recipes-test-repeats-coverage
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: card7-budget-pass-repeats-card6
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 is the first round; no earlier ledger exists.
F4 (`card8-unreachable-profile-recipe-fallback`) is the only gating finding.
Its fix replaces a fallback sentence and drops one test row, so the result is fixed by the finding text.

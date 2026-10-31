---
round: 1
ledger:
  - key: card5-registry-before-recipe-reach
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: landingshed-deps-docs-stale-after-entry-gate
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: session-spec-asserts-stencil-prose
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: handback-file-comment-untargeted
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: idle-timeout-template-comment-stale
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: rownames-doc-stale
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: card9-impactsummary-vs-intent
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 is the first round, so there is no earlier ledger.
The blocking defect is sequencing: card 5 registers the sampler engine before card 6 makes `RecipeEngines` reach it.
The other gating item is a test-economy defect in card 3.
The remaining five are stale-doc or internal-disagreement findings with local fixes.

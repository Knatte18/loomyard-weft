---
round: 2
ledger:
  - key: card5-registry-before-recipe-reach
    rounds: [1]
    status: resolved
  - key: landingshed-deps-docs-stale-after-entry-gate
    rounds: [1]
    status: resolved
  - key: session-spec-asserts-stencil-prose
    rounds: [1]
    status: resolved
  - key: handback-file-comment-untargeted
    rounds: [1]
    status: resolved
  - key: idle-timeout-template-comment-stale
    rounds: [1]
    status: resolved
  - key: rownames-doc-stale
    rounds: [1]
    status: resolved
  - key: card9-impactsummary-vs-intent
    rounds: [1, 2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: card5-testnames-pin-missing
    rounds: [2]
    status: open
    class: scope
    severity: BLOCKING
    fix: mechanical
  - key: amendmenttarget-doc-stale
    rounds: [2]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: childseedparams-handback-docs-stale
    rounds: [2]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: card3-stencil-bare-citation
    rounds: [2]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
---

Round 1's seven keys are all carried.
Six are resolved: round 2 raises none of them again, and it states that card 5's registry-before-recipe ordering and card 3's test economy are settled in the plan's favour.
The round 1 ImpactSummary key stays open because round 2's F5 is the same defect on other cards.
Round 2's blocking item is the registry name pin that card 5 does not update.
The remaining new items are stale docs on untargeted symbols (`AmendmentTarget`, `childSeedParams`, `InnerRunDeps`) and the stencil's bare `docs/` path.

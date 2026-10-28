---
round: 2
ledger:
  - key: d4-resolves-through-missing-index-method
    rounds: [1]
    status: resolved
  - key: d5-scan-trigger-misses-reflect-type-and-visiblefields
    rounds: [1]
    status: resolved
  - key: d8-test-member-glyph-classification
    rounds: [1]
    status: resolved
  - key: uses-listing-escape-hatch-unbounded
    rounds: [1]
    status: resolved
  - key: d9-marker-source-and-two-builders
    rounds: [1]
    status: resolved
  - key: d2-check-order-vs-darn-early-return
    rounds: [1]
    status: resolved
  - key: d1-findings-no-way-forward
    rounds: [1]
    status: resolved
  - key: d3-root-entry-reads-untracked
    rounds: [1]
    status: resolved
  - key: d3-go-package-pattern-as-path
    rounds: [1]
    status: resolved
  - key: d5-unqualified-nilseams-in-envkit
    rounds: [1]
    status: resolved
  - key: d7-tier-keys-vs-build-tags
    rounds: [1]
    status: resolved
  - key: d10-pointer-renders-twice-in-chair
    rounds: [1]
    status: resolved
  - key: d5-acceptance-omits-marker-glyph-refusal
    rounds: [1]
    status: resolved
  - key: d3-placeholder-exemption-unbounded
    rounds: [1]
    status: resolved
  - key: d6-widening-not-in-auto-mode-assumptions
    rounds: [1]
    status: resolved
  - key: d10-reads-citation-rules-as-darn
    rounds: [1]
    status: resolved
  - key: d5-marker-set-misses-nested-nilseams-structs
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d4-struct-field-citation-has-no-glyph
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d6-later-card-cover-unbounded
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: d4-unpartitioned-ambiguous-and-rejected-outcomes
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: d8-rename-and-helper-test-members-misclassified
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d6-test-file-cover-collides-with-edit-test-file
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d5-bound-omits-same-package-non-test-helpers
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d9-index-scoped-to-d8-though-d6-needs-it
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d4-member-shape-admits-placeholder-token
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d4-markdown-anchor-read-as-glyph
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: d7-relaxation-missing-from-auto-mode-assumptions
    rounds: [2]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1's sixteen keys were not raised again by round 2's fresh review and are carried as resolved.
Round 2 raised eleven new keys, F1 to F11 in review order.
Three are MEDIUM design (nested `NilSeams` marker set, struct-field citations without a glyph, unbounded later-card cover in D6); they sit on keys first raised in round 2, so the run converges and Go carries them into Open risks.

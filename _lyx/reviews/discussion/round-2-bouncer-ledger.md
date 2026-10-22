---
round: 2
ledger:
  - key: interface-method-misfires-on-csharp
    rounds: [1]
    status: resolved
  - key: resign-minted-id-reaches-no-later-card
    rounds: [1]
    status: resolved
  - key: rename-derivation-still-go-string-work
    rounds: [1]
    status: resolved
  - key: d4-misses-go-parsable-csharp-plan
    rounds: [1]
    status: resolved
  - key: namespace-pair-passes-as-file-pair
    rounds: [1]
    status: resolved
  - key: d3-bound-omits-checks-and-scopeguard
    rounds: [1]
    status: resolved
  - key: d4-go-reading-contradicts-d3-constraints
    rounds: [1]
    status: resolved
  - key: relaxed-resign-guard-has-no-bound
    rounds: [1]
    status: resolved
  - key: explicit-interface-member-owner
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: listing-rule-governs-no-reader
    rounds: [1]
    status: resolved
  - key: d1-resign-lang-source
    rounds: [1]
    status: resolved
  - key: partitioning-two-rules
    rounds: [1]
    status: resolved
  - key: verify-package-rationale-false
    rounds: [1]
    status: resolved
  - key: requirement-in-optional-notes
    rounds: [1]
    status: resolved
  - key: scope-paragraph-and-open-risk-repeat
    rounds: [1]
    status: resolved
  - key: rename-multi-declarator-field
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: missing-root-self-glyph-open-risk
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d4-bound-misstates-namespace-glyph-under-go
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: resign-batch-order-vs-driftcheck-scopeguard
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: overload-collision-misses-plan-declared-handle
    rounds: [2]
    status: open
    class: scope
    severity: LOW
    fix: design-wrong
---

Round 1 raised fifteen findings, three BLOCKING.
Round 2 has no BLOCKING finding, and all round-1 keys but one have no matching finding, so they are resolved.
The qualifier problem on explicit interface members (`explicit-interface-member-owner`) remains: round 1's fix reached only the handle bullet, and the re-sign arrow and Rename still keep the qualifier in `Owner`.
Round 2 adds two design MEDIUM keys (multi-declarator field Rename, deferred root-self-glyph open risk), one consistency LOW on D4's bound, one design LOW on re-sign batch ordering, and one scope LOW on overload collision.

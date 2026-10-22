---
round: 3
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
    rounds: [1, 2, 3]
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
    status: resolved
  - key: missing-root-self-glyph-open-risk
    rounds: [2]
    status: resolved
  - key: d4-bound-misstates-namespace-glyph-under-go
    rounds: [2, 3]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: resign-batch-order-vs-driftcheck-scopeguard
    rounds: [2, 3]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: overload-collision-misses-plan-declared-handle
    rounds: [2, 3]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: field-head-cut-ignores-generics-tuples
    rounds: [3]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: open-risk-cs-scan-trigger-understated
    rounds: [3]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised fifteen findings, three BLOCKING; round 2 resolved all but the interface-qualifier key.
Round 3 has no BLOCKING finding.
The multi-declarator field head and root-self-glyph open risk from round 2 are resolved, but the field head rule now fails on generic and tuple types (new key).
`explicit-interface-member-owner` is open a third time: the owner chain and its interface qualifier are stated in two bullets, and the test for which trailing components are the qualifier is undefined.
The re-sign batch-order key is reopened on the plan-fingerprint restamp and the record-batch/recover-batch shared pass.
Remaining items are a collision form (two re-signs minting one id), D4's bound wording and the scan-trigger wording in the third Open risk.

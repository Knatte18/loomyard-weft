---
round: 1
ledger:
  - key: interface-method-misfires-on-csharp
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: resign-minted-id-reaches-no-later-card
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: rename-derivation-still-go-string-work
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: d4-misses-go-parsable-csharp-plan
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: namespace-pair-passes-as-file-pair
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d3-bound-omits-checks-and-scopeguard
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d4-go-reading-contradicts-d3-constraints
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: relaxed-resign-guard-has-no-bound
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: explicit-interface-member-owner
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: listing-rule-governs-no-reader
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d1-resign-lang-source
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: partitioning-two-rules
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: verify-package-rationale-false
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: requirement-in-optional-notes
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: scope-paragraph-and-open-risk-repeat
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 is the first round; no earlier ledger exists.
Three BLOCKING design findings sit on D5 and D7: `isInterfaceMethod` misfires on every C# method, a C# re-sign's minted id reaches no later card, and C# Rename derivation still runs Go string work.
Three MEDIUM design findings cover D3 and D4 gaps: a Go-parsable C# plan escapes D4, a namespace pair passes as a file pair, and D3's bound omits checks and `ScopeGuard`.
One MEDIUM decision finding asks for a bound on the relaxed re-sign guard, and one MEDIUM consistency finding asks to reconcile D4's Go reading with D3 and the Constraints.
The six LOW findings and one NIT are local corrections and relocations.

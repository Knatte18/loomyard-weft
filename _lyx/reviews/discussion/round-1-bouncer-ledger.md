---
round: 1
ledger:
  - key: warm-node-stdio-unresearched
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: empty-test-scope-hides-compile-break
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: empty-scope-build-holds-no-slot
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: build-failure-read-as-test-failure
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: inherited-test-methods-not-selected
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: bound-omits-polymorphic-dispatch
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: carry-false-skipped-outside-green-record
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: vstest-gated-project-assumed
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: mtp-zero-tests-risk-false-premise
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: inert-only-vs-carry-false-contradiction
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: tier1-dotnet-test-needs-network
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: filter-length-limit-unnamed
    rounds: [1]
    status: open
    class: decision
    severity: LOW
    fix: mechanical
  - key: new-fingerprint-fields-void-green-records
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: empty-diff-under-dotnet-undefined
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: filter-rendering-undefined-for-go-scopes
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 maps F1 to F15 in order onto the keys above.
Nine are gating `design` findings at MEDIUM: unresearched facts (F1, F8, F9), scope-derivation gaps (F2, F5, F6), build-failure reporting (F4), the `carry: false` claim (F7) and the network-dependent tier1 test (F11).
The rest are consistency or wording fixes.

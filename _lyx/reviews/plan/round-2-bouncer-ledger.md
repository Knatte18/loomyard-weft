---
round: 2
ledger:
  - key: declared-name-index-fallback-wrong-token
    rounds: [1]
    status: resolved
  - key: spaced-interface-qualifier-cannot-bind
    rounds: [1]
    status: resolved
  - key: scope-guard-go-row-reports-finding
    rounds: [1]
    status: resolved
  - key: language-mismatch-evidence-too-narrow
    rounds: [1]
    status: resolved
  - key: create-fallback-lang-source-unnamed
    rounds: [1]
    status: resolved
  - key: csharp-create-body-no-way-forward
    rounds: [1]
    status: resolved
  - key: resign-head-mismatch-way-forward-misapplied
    rounds: [1]
    status: resolved
  - key: cross-alphabet-detail-constructed-path
    rounds: [1]
    status: resolved
  - key: fabric-vocabulary-go-package-dir-unguarded
    rounds: [1]
    status: resolved
  - key: caller-coverage-gate-no-discriminating-row
    rounds: [1]
    status: resolved
  - key: resign-not-done-placeholder-batch-number
    rounds: [1]
    status: resolved
  - key: rename-namespace-pair-move-after-create-missing
    rounds: [1]
    status: resolved
  - key: boundary-go-file-header-stale
    rounds: [1]
    status: resolved
  - key: resolve-half-refuses-handle-only-go-plan
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: invalid-container-row-no-way-forward
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: csharp-owner-indexer-qualifier-left-in-owner
    rounds: [2]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
---

Round 1 raised 13 findings, three BLOCKING, on card 9's quarry name binding, card 4's fixture and card 6's evidence.
Round 2 did not re-raise any of them, so all are resolved.
Round 2 raised three new findings: F1 (card 6's resolve half refuses a Go plan whose Go work is only handles, design MEDIUM), F2 (card 9's `invalid_container` row names no way forward, LOW) and F3 (`csharpOwner` leaves an explicit-interface indexer's qualifier in the owner, LOW).
F1 is gating-class and first raised in round 2, so it goes to Open risks.

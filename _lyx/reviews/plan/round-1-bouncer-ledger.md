---
round: 1
ledger:
  - key: declared-name-index-fallback-wrong-token
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: spaced-interface-qualifier-cannot-bind
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: scope-guard-go-row-reports-finding
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: language-mismatch-evidence-too-narrow
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: create-fallback-lang-source-unnamed
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: csharp-create-body-no-way-forward
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: resign-head-mismatch-way-forward-misapplied
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: cross-alphabet-detail-constructed-path
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: fabric-vocabulary-go-package-dir-unguarded
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: caller-coverage-gate-no-discriminating-row
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: resign-not-done-placeholder-batch-number
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: rename-namespace-pair-move-after-create-missing
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: boundary-go-file-header-stale
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 (F1 to F13, in the order above) is the first review of the plan.
Gating: F1 to F3 BLOCKING, F4, F6, F7, F8 design MEDIUM.
F1 to F2 rest on wrong claims about quarry v0.4.0 name binding in card 9; F3 is a card 4 fixture whose go row reports the finding it claims absent.

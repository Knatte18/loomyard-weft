---
round: 1
ledger:
  - key: stencil-measuring-route-names-tier
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
    fix: mechanical
  - key: tierless-gate-skips-lints
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: stencil-no-test-contradicts-gate-line
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: allscope-fallback-branches-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: gate-test-help-scope-all-stale
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: gate-template-comments-stale
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: darn-docs-unconditional-no-gate
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: dotnet-hub-parent-align-whole-run
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised eight findings, F1 to F8, in the order of the keys above.
The gating ones are F1 (BLOCKING, `PATTERN-test-speed`) and F2 (MEDIUM design).
No earlier ledger exists, so nothing carries forward.

---
round: 2
ledger:
  - key: warm-node-stdio-unresearched
    rounds: [1]
    status: resolved
  - key: empty-test-scope-hides-compile-break
    rounds: [1]
    status: resolved
  - key: empty-scope-build-holds-no-slot
    rounds: [1]
    status: resolved
  - key: build-failure-read-as-test-failure
    rounds: [1]
    status: resolved
  - key: inherited-test-methods-not-selected
    rounds: [1]
    status: resolved
  - key: bound-omits-polymorphic-dispatch
    rounds: [1]
    status: resolved
  - key: carry-false-skipped-outside-green-record
    rounds: [1]
    status: resolved
  - key: vstest-gated-project-assumed
    rounds: [1]
    status: resolved
  - key: mtp-zero-tests-risk-false-premise
    rounds: [1]
    status: resolved
  - key: inert-only-vs-carry-false-contradiction
    rounds: [1]
    status: resolved
  - key: tier1-dotnet-test-needs-network
    rounds: [1]
    status: resolved
  - key: filter-length-limit-unnamed
    rounds: [1]
    status: resolved
  - key: new-fingerprint-fields-void-green-records
    rounds: [1]
    status: resolved
  - key: empty-diff-under-dotnet-undefined
    rounds: [1]
    status: resolved
  - key: filter-rendering-undefined-for-go-scopes
    rounds: [1]
    status: resolved
  - key: card-seed-misses-type-glyphs-and-inert-targets
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: build-not-an-open-config-map
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: tier-filter-key-has-no-bound
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: lexer-interpolation-holes-break-bound
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: filter-term-spelling-unstated
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: build-failure-exit-code-depends-on-plan-order
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: empty-diff-carry-false-opted-in-tier
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: tier-not-opted-in-under-empty-test-scope
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: filter-rendering-keyed-on-call-not-tier
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: selected-class-without-single-ancestor-project
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: untracked-files-skip-inert-check
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: open-risk-understates-wide-closure-cost
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: tier1-fixture-may-never-start-node
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: q2-contradicts-decision-10
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: one-whole-run-per-task-is-conditional
    rounds: [2]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1's fifteen keys are not raised again in round 2 and are recorded resolved.
Round 2 raised fifteen new keys, F1 to F15 in order.
Five are gating `design` findings at MEDIUM: the card seed (F1), the `build` config shape (F2), the filter bound (F3), the lexer's interpolation holes (F4) and the filter term spelling (F5).
The rest are LOW consistency or design gaps and one NIT.

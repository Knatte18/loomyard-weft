---
round: 1
ledger:
  - key: hubpaths-literal-bolt-wants
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
    fix: mechanical
  - key: hubpaths-duplicates-hubscratch-test
    rounds: [1]
    status: open
    class: scope
    severity: BLOCKING
    fix: mechanical
  - key: unreached-production-functions-unlicensed
    rounds: [1]
    status: open
    class: decision
    severity: BLOCKING
    fix: design-wrong
  - key: runrule-no-lock-seam
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: read-failure-contract-unspecified
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
    fix: design-wrong
  - key: core-store-literals-redeclared
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
    fix: mechanical
  - key: snapshot-backoffdelays-no-supplier
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: records-layout-row-reads-cache
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: recordingtb-nil-embed-panics
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: refusal-error-allowlist-one-card-late
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: moved-name-tests-left-in-engine
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: fabric-test-code-grows
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: mechanical
  - key: prose-naming-moved-home-missed
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: snapshot-two-meanings-in-docs
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 opened every key.
The blocking defects sit in two places: card 3's `TestHubPaths`,
and the core, executor loop and snapshot reader of cards 12-15.
`unreached-production-functions-unlicensed` depends on a parent decision,
and `refusal-error-allowlist-one-card-late` depends on how that decision falls.

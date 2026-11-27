---
round: 2
ledger:
  - key: hubpaths-literal-bolt-wants
    rounds: [1]
    status: resolved
  - key: hubpaths-duplicates-hubscratch-test
    rounds: [1]
    status: resolved
  - key: unreached-production-functions-unlicensed
    rounds: [1]
    status: resolved
  - key: runrule-no-lock-seam
    rounds: [1]
    status: resolved
  - key: read-failure-contract-unspecified
    rounds: [1, 2]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: core-store-literals-redeclared
    rounds: [1]
    status: resolved
  - key: snapshot-backoffdelays-no-supplier
    rounds: [1]
    status: resolved
  - key: records-layout-row-reads-cache
    rounds: [1]
    status: resolved
  - key: recordingtb-nil-embed-panics
    rounds: [1]
    status: resolved
  - key: refusal-error-allowlist-one-card-late
    rounds: [1]
    status: resolved
  - key: moved-name-tests-left-in-engine
    rounds: [1]
    status: resolved
  - key: fabric-test-code-grows
    rounds: [1]
    status: resolved
  - key: prose-naming-moved-home-missed
    rounds: [1]
    status: resolved
  - key: snapshot-two-meanings-in-docs
    rounds: [1]
    status: resolved
  - key: net-shrink-left-to-card-15-halt
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
---

Round 1's defects sat in card 3's `TestHubPaths` and in the core, executor loop and snapshot reader of cards 12-15.
Round 2 resolves all of them but `read-failure-contract-unspecified`,
now narrowed to store order, per-field `Store` values and which phase a failed read records.

`fabric-test-code-grows` was settled by the added entry "This part's fabric test code shrinks in net".
`net-shrink-left-to-card-15-halt` is its successor: whether the plan meets that entry.
The reviewer estimates the planned deletions fall short of the planned additions,
and its fix allows going to the parent for an amended entry when no further deletions exist.

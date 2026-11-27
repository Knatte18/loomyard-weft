---
round: 3
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
    status: resolved
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
    rounds: [2, 3]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: executor-tests-fake-git-errors
    rounds: [3]
    status: open
    class: consistency
    severity: BLOCKING
    fix: mechanical
---

Round 1's defects sat in card 3's `TestHubPaths` and in the core, executor loop and snapshot reader of cards 12-15.
Round 2 left the read-failure contract and the net shrink of fabric test code open.
Round 3 resolves the read-failure contract.

`net-shrink-left-to-card-15-halt` has narrowed: the plan now shows the shrink and has no halt step.
What remains is that some of card 15's deletion reasons do not hold, one of them leaving a subpath-anchor check of `SeedWarpExclude` unheld.

`executor-tests-fake-git-errors` is new: card 14's tier0 tests hand-write `gitexec` and `gitrepo` errors,
which `PATTERN-test-stand-ins` bars.

`fabric-test-code-grows` was settled by the added entry "This part's fabric test code shrinks in net".

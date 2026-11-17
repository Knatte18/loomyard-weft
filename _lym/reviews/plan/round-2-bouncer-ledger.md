---
round: 2
ledger:
  - key: card1-linked-packages-lose-their-maps
    rounds: [1]
    status: resolved
  - key: card1-scope-subtests-cannot-narrow
    rounds: [1]
    status: resolved
  - key: card3-guard-kill-beside-another-unreachable
    rounds: [1]
    status: resolved
  - key: card1-badbase-refusal-unreachable
    rounds: [1]
    status: resolved
  - key: card1-package-set-whole-cases
    rounds: [1]
    status: resolved
  - key: card1-removed-guards-no-bound
    rounds: [1]
    status: resolved
  - key: card1-testcover-unreachable-until-card2
    rounds: [1]
    status: resolved
  - key: card3-misses-two-exit1-subtests
    rounds: [1]
    status: resolved
  - key: card3-tier0-tests-name-no-file
    rounds: [1]
    status: resolved
  - key: card2-doc-names-deleted-goroot
    rounds: [1]
    status: resolved
  - key: card1-second-left-out-reason-unreached
    rounds: [1]
    status: resolved
  - key: card3-collides-timing-margin
    rounds: [1]
    status: resolved
  - key: card4-write-after-run-failure-unreached
    rounds: [1]
    status: resolved
  - key: card1-stale-selector-header-mention
    rounds: [1]
    status: resolved
  - key: card1-tryacquire-profiledfuncs-test-only
    rounds: [2]
    status: open
    class: scope
    severity: BLOCKING
    fix: mechanical
  - key: card1-computescope-green-base
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
---

Round 1 raised fourteen findings; round 2's review does not repeat any of them, so each is resolved.
Round 2 raised two new keys.
`card1-tryacquire-profiledfuncs-test-only` is the F1 finding: card 1 strands `gateslot.Pool.TryAcquire` and `testcover.FuncIndex.ProfiledFuncs` as test-only code.
`card1-computescope-green-base` is the F2 finding: card 1 passes the green record to `computeScope`, so a clean tree after a green tier0 run maps no package.

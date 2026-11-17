---
round: 3
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
    status: resolved
  - key: card1-computescope-green-base
    rounds: [2]
    status: resolved
  - key: card1-inert-field-write-only
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: card3-mapsidebyside-parallel-fallback
    rounds: [3]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised fourteen findings and round 2 raised two more; round 3's review repeats none of them, so all are resolved.
Round 3 raised two new LOW keys.
`card1-inert-field-write-only` (F1): card 1 deletes `headSelector`, the only reader of `testMapRun.inert`, but leaves the field and its assignment in `prepare`.
`card3-mapsidebyside-parallel-fallback` (F2): card 3's `mapSideBySide` falls back to `runtime.NumCPU()` for a tier `Parallel` below 1, which the gate config refuses, and a `TestMapSideBySide` row pins it.
Both fixes are local text edits the findings already specify.

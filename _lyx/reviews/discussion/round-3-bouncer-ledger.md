---
round: 3
ledger:
  - key: d2-refusal-stale-verify-record
    rounds: [1, 2]
    status: resolved
  - key: d2-bound-no-retry-loss
    rounds: [1]
    status: resolved
  - key: d4-target-file-match-and-bound
    rounds: [1, 2]
    status: resolved
  - key: d4-pr-narrative-wrong-stencil
    rounds: [1]
    status: resolved
  - key: d5-landing-order-source
    rounds: [1, 2]
    status: resolved
  - key: d5-bound-semantic-overlap
    rounds: [1]
    status: resolved
  - key: d1-pause-check-hand-cleared
    rounds: [1]
    status: resolved
  - key: d5-parent-commit-design-conflict
    rounds: [2]
    status: resolved
  - key: d4-digest-drops-symbol-names
    rounds: [2]
    status: resolved
  - key: d4-external-test-unit
    rounds: [3]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d5-darn-peer-row-source
    rounds: [3]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: d5-parent-commit-duplicate-case
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised seven findings, round 2 five, round 3 three.
Every earlier key is resolved: round 3 raised none of them, and the review lists D2's Bound, D5's row source, the parent-commit comparison and D4's reader-visible loss as settled.
Round 3 opens three new keys.
`d4-external-test-unit` is the only gating one: a Go symbol target in an external test package has the `<dir>_test` pseudo-unit, which `Glyph.UnitPath` alone does not map to a directory.
`d5-darn-peer-row-source` asks which recipe a darn peer's row is read against.
`d5-parent-commit-duplicate-case` asks whether a landed commit that already makes one of this run's decisions should escalate, matching the peer rule.

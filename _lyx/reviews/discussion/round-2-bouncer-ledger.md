---
round: 2
ledger:
  - key: d2-refusal-stale-verify-record
    rounds: [1, 2]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: d2-bound-no-retry-loss
    rounds: [1]
    status: resolved
  - key: d4-target-file-match-and-bound
    rounds: [1, 2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: d4-pr-narrative-wrong-stencil
    rounds: [1]
    status: resolved
  - key: d5-landing-order-source
    rounds: [1, 2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: d5-bound-semantic-overlap
    rounds: [1]
    status: resolved
  - key: d1-pause-check-hand-cleared
    rounds: [1]
    status: resolved
  - key: d5-parent-commit-design-conflict
    rounds: [2]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: d4-digest-drops-symbol-names
    rounds: [2]
    status: open
    class: decision
    severity: LOW
    fix: mechanical
---

Round 1 raised seven findings; round 2 raised five.
Four round-1 keys (`d2-bound-no-retry-loss`, `d4-pr-narrative-wrong-stencil`, `d5-bound-semantic-overlap`, `d1-pause-check-hand-cleared`) did not recur and are resolved; round 2 explicitly cleared D1.
`d4-target-file-match-and-bound` reopened as BLOCKING: a Go symbol target's glyph unit is a package directory, so the file-level exemption needs an explicit per-shape rule.
`d5-landing-order-source` reopened as BLOCKING: no hub loom recipe file exists, so the row order needs a surface lyx serves.
`d2-refusal-stale-verify-record` continues as a MEDIUM consistency finding: D2 narrows the Publish refusal, which its Bound denies.
New this round: D5's treatment of design conflicts with landed parent commits, and D4 dropping the guard's symbol names from the digest.

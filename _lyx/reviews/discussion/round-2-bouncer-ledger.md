---
round: 2
ledger:
  - key: worktree-former-value-drop-marker-gated
    rounds: [1]
    status: resolved
  - key: older-binary-refill-unbounded-no-repair
    rounds: [1, 2]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: d1-review-guard-weakening-no-bound
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: sparse-base-invariant-no-pattern-entry
    rounds: [1]
    status: resolved
  - key: prime-seed-drops-former-values-after-marker
    rounds: [1]
    status: resolved
  - key: key-move-into-sparse-base-rules-open
    rounds: [1]
    status: resolved
  - key: deliberate-pre-deploy-value-dropped-unassumed
    rounds: [1]
    status: resolved
  - key: batcher-profiles-whole-map-misses-hubs
    rounds: [1]
    status: resolved
  - key: editor-scaffolded-base-no-header
    rounds: [1]
    status: resolved
  - key: key-move-drops-template-equal-outside-walk
    rounds: [2]
    status: open
    class: consistency
    severity: BLOCKING
    fix: mechanical
  - key: key-move-empties-source-base-deleted
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: worktree-former-drop-changes-resolved-value
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: options-pair-former-drop-no-criterion
    rounds: [2]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
---

Round 1's nine keys: five are resolved, since round 2 raises nothing on them (worktree former-value gate, PATTERN entry, prime seed, deliberate pre-deploy value, batcher whole-map, editor header, and the key-move rules as first stated).
Two carry on: the D1 guard bound is still incomplete (reviewer and fixer downgrades, BLOCKING auto-accept, Merge-Review as a bound), and the older-binary refill list still omits the clone path.
Round 2's new findings are the BLOCKING key-move drop outside the walk, the emptied key-move source base, the worktree drop's resolved-value change and the missing `Options.Pair` criterion.

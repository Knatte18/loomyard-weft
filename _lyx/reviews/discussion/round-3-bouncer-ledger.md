---
round: 3
ledger:
  - key: worktree-former-value-drop-marker-gated
    rounds: [1]
    status: resolved
  - key: older-binary-refill-unbounded-no-repair
    rounds: [1, 2]
    status: resolved
  - key: d1-review-guard-weakening-no-bound
    rounds: [1, 2, 3]
    status: open
    class: design
    severity: LOW
    fix: mechanical
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
    status: resolved
  - key: key-move-empties-source-base-deleted
    rounds: [2]
    status: resolved
  - key: worktree-former-drop-changes-resolved-value
    rounds: [2]
    status: resolved
  - key: options-pair-former-drop-no-criterion
    rounds: [2]
    status: resolved
  - key: clone-marker-contradicts-marker-precondition
    rounds: [3]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: prime-seed-drop-contradicts-strip-nothing-callers
    rounds: [3]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: failed-walk-criterion-exceeds-undo
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: strip-emptied-base-not-kept-present
    rounds: [3]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: move-into-absent-target-skips-prime-seed
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 and 2 keys that round 3 does not raise again are resolved, including the older-binary refill list, which round 3 judged complete, and the key-move source base, worktree former-value drop and `Options.Pair` keys.
The D1 guard bound remains open at LOW: the driver model downgrade is the last item missing from its list.
Round 3's new keys are the clone marker precondition, the prime seed against the strip-nothing callers, the failed-walk criterion, the strip emptying a base, and the move into an absent target; none is gating at MEDIUM or worse.

---
round: 1
ledger:
  - key: worktree-former-value-drop-marker-gated
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: older-binary-refill-unbounded-no-repair
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: d1-review-guard-weakening-no-bound
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: mechanical
  - key: sparse-base-invariant-no-pattern-entry
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: prime-seed-drops-former-values-after-marker
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: key-move-into-sparse-base-rules-open
    rounds: [1]
    status: open
    class: decision
    severity: LOW
    fix: design-wrong
  - key: deliberate-pre-deploy-value-dropped-unassumed
    rounds: [1]
    status: open
    class: decision
    severity: LOW
    fix: mechanical
  - key: batcher-profiles-whole-map-misses-hubs
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: editor-scaffolded-base-no-header
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised nine findings; all stay open.
The BLOCKING one is the board-marker gate on the worktree former-value drop.
The other gating-class finding is the missing PATTERN entry for the sparse-base write rule, which the task's declared PATTERN changes now list.
The older-binary refill, the unbounded D1 guard weakening, the key-move rules and the batcher whole-map miss are bound or repair gaps in the decision record.
The prime-seed marker gate, the deliberate pre-deploy value and the editor header are local corrections.

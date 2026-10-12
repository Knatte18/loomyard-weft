---
round: 3
ledger:
  - key: anchor-lyx-probe-breaks-hubwide-load
    rounds: [1]
    status: resolved
  - key: migration-strip-stacks-layers-outside-configengine
    rounds: [1, 2]
    status: resolved
  - key: new-hubreconcile-test-instead-of-extending
    rounds: [1]
    status: resolved
  - key: retired-key-way-forward-misses-repo-default
    rounds: [1]
    status: resolved
  - key: loader-cards-duplicate-tests
    rounds: [1, 2]
    status: resolved
  - key: d6-seeding-escapes-hubreconcile-ensure
    rounds: [1]
    status: resolved
  - key: outputsuccesswithtask-missing-resign-arrow
    rounds: [1]
    status: resolved
  - key: seedchild-stuck-route-tested-only-via-stub
    rounds: [1]
    status: resolved
  - key: three-guard-branches-without-named-test
    rounds: [1]
    status: resolved
  - key: docs-constraint-misses-configsync-and-walk-commit
    rounds: [1]
    status: resolved
  - key: layer-stacking-outside-configengine-in-seed-only-readers
    rounds: [2]
    status: resolved
  - key: fresh-hub-has-no-per-worktree-repo-default
    rounds: [2]
    status: resolved
  - key: absent-base-refusal-way-forward-cannot-clear-per-worktree
    rounds: [2]
    status: resolved
  - key: failed-walk-leaves-sparse-marker
    rounds: [2]
    status: resolved
  - key: d5-fill-log-carries-only-template-layer
    rounds: [2]
    status: resolved
  - key: card16-walk-assertions-on-migrated-worktrees
    rounds: [3]
    status: open
    class: design
    severity: BLOCKING
  - key: clone-outcome-after-card-16-unreflected
    rounds: [3]
    status: open
    class: scope
    severity: MEDIUM
---

Rounds 1 and 2 raised sixteen findings between them, and round 3 reports no defect on any of their keys, so all fifteen earlier keys are resolved.

Round 3 raises two new findings, both about the interaction of cards 15 and 16:

- `card16-walk-assertions-on-migrated-worktrees` (F1): after card 15, a clone's `Ensure` walk already seeds defaults and, with card 16, commits the sparse marker in the prime, and the pair inherits it.
  Card 16's three `Ensure` integration assertions therefore start from marked worktrees with nothing to strip and cannot hold.
  The proposed fix makes the `newStaleHub` fixture write full pre-migration files and delete the marker.
- `clone-outcome-after-card-16-unreflected` (F2): the hubforge seeding contract, `CloneAndWire`'s mutation record and the clean-hub scenario's record-shape step do not follow card 16's change to what a clone produces.

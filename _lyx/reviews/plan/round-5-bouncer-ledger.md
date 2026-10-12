---
round: 5
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
    status: resolved
  - key: clone-outcome-after-card-16-unreflected
    rounds: [3]
    status: resolved
  - key: clone-output-tests-unplanned-after-card-15
    rounds: [4, 5]
    status: open
    class: design
    severity: BLOCKING
  - key: card16-newstalehub-order-against-seedretiredkey
    rounds: [4]
    status: resolved
  - key: merge-record-uncleared-before-next-walk-assertion
    rounds: [4]
    status: resolved
  - key: second-ensure-meets-current-stamp-no-walk
    rounds: [4]
    status: resolved
  - key: reseed-case-collides-with-newstalehub-migration
    rounds: [5]
    status: open
    class: consistency
    severity: MEDIUM
  - key: hubreconcile-test-edits-not-member-glyphs
    rounds: [5]
    status: open
    class: consistency
    severity: MEDIUM
---

Rounds 1 to 3 raised eighteen findings, all resolved by round 4.
Round 4's three consistency keys draw no finding in round 5 and are resolved.

The design key `clone-output-tests-unplanned-after-card-15` recurs: round 4 named four tests, round 5 names three more (`TestRunCLI_CloneEndToEnd`, `TestRunCLI_AnchoredHubScenario`'s anchor-scoped step, and the pair-inherits step of `TestRunCLI_CleanHubScenario`).
The fix asked for is the sweep method, not another list: name the search that finds every test reading a per-worktree file from a hubforge hub or clone without seeding it.

Round 5 adds two consistency keys:

- `reseed-case-collides-with-newstalehub-migration`: card 15's reseed case, under card 16's fixture, strips and deletes the reseeded module's files, and `batcher` would break `retiredKeyPresent`.
- `hubreconcile-test-edits-not-member-glyphs`: cards 15 and 16 list a bare test file or omit the changed helper `runStaleBuildReconcile`.

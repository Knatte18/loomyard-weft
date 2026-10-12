---
round: 4
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
    rounds: [4]
    status: open
    class: design
    severity: BLOCKING
  - key: card16-newstalehub-order-against-seedretiredkey
    rounds: [4]
    status: open
    class: consistency
    severity: MEDIUM
  - key: merge-record-uncleared-before-next-walk-assertion
    rounds: [4]
    status: open
    class: consistency
    severity: LOW
  - key: second-ensure-meets-current-stamp-no-walk
    rounds: [4]
    status: open
    class: consistency
    severity: LOW
---

Rounds 1 to 3 raised eighteen findings, all resolved by round 4.
Round 3's two open keys (card 16's walk assertions on migrated worktrees, clone outcome unreflected) draw no finding in round 4.

Round 4 raises four new keys:

- `clone-output-tests-unplanned-after-card-15` (F1, BLOCKING design): card 15 stops a clone writing per-worktree files into the prime, but `TestNewHub`, two `TestRunCLI_CleanHubScenario` steps and the `hub config reconcile` subtest of `TestOrchIntegration_Refusals` still expect them.
- `card16-newstalehub-order-against-seedretiredkey` (F2, MEDIUM): the order of `newStaleHub`'s full file set against `seedRetiredKey` is open, and either order breaks the retired-key assertions.
- `merge-record-uncleared-before-next-walk-assertion` (F3, LOW): the restore test's merge record is never cleared, so its next-walk assertion cannot run.
- `second-ensure-meets-current-stamp-no-walk` (F4, LOW): the second `Ensure` meets a current stamp, so no walk runs.

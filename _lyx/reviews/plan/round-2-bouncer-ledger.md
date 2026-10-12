---
round: 2
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
    status: open
    class: consistency
    severity: BLOCKING
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
    status: open
    class: design
    severity: BLOCKING
  - key: fresh-hub-has-no-per-worktree-repo-default
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
  - key: absent-base-refusal-way-forward-cannot-clear-per-worktree
    rounds: [2]
    status: open
    class: consistency
    severity: BLOCKING
  - key: failed-walk-leaves-sparse-marker
    rounds: [2]
    status: open
    class: scope
    severity: MEDIUM
  - key: d5-fill-log-carries-only-template-layer
    rounds: [2]
    status: open
    class: decision
    severity: LOW
---

Round 1 raised ten findings.
Round 2 does not repeat eight of them; they are resolved.

The two that carry on:

- `migration-strip-stacks-layers-outside-configengine` (round 1 F2) was fixed for card 16's configsync, but the same split reappears as round 2 F3 in cards 7 and 8.
  It is tracked under the new key `layer-stacking-outside-configengine-in-seed-only-readers`, and the round 1 key is resolved.
- `loader-cards-duplicate-tests` (round 1 F5) recurs as round 2 F4: the trimmed loader rows still use keys card 13's table pins.
  Round 2 labels it `consistency` BLOCKING.

New in round 2:

- F1: no route seeds a per-worktree repo default in a fresh hub before card 9's strict load.
- F2: card 2's absent-base refusal names a verb that leaves a per-worktree default absent.
- F5: the sparse marker survives a failed walk.
- F6: D5's per-layer fill log is carried only for the template layer.

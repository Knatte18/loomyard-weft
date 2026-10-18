---
round: 2
ledger:
  - key: command-keys-settable-from-task-branch
    rounds: [1]
    status: resolved
  - key: remove-lock-inside-deleted-tree
    rounds: [1]
    status: resolved
  - key: local-remove-guard-makes-force-routine
    rounds: [1]
    status: resolved
  - key: late-child-recreates-removed-worktree-dir
    rounds: [1]
    status: resolved
  - key: absorbing-lock-premise-false
    rounds: [1]
    status: resolved
  - key: remove-push-lock-wait-unbounded
    rounds: [1]
    status: resolved
  - key: noop-transition-skip-not-existing
    rounds: [1]
    status: resolved
  - key: detached-child-loses-pause-slug
    rounds: [1]
    status: resolved
  - key: force-contradicts-warp-branch-gate
    rounds: [1]
    status: resolved
  - key: guard-order-vs-pending-records-commit
    rounds: [1]
    status: resolved
  - key: cleanup-publishes-local-orphan-weft
    rounds: [1]
    status: resolved
  - key: batten-env-setting-unwired
    rounds: [1]
    status: resolved
  - key: under-100ms-criterion-vs-d2-cut
    rounds: [1]
    status: resolved
  - key: window-override-no-mechanism
    rounds: [1]
    status: resolved
  - key: cmd-percent-expansion-in-quotes
    rounds: [1]
    status: resolved
  - key: half-marked-pair-remove-undefined
    rounds: [1]
    status: resolved
  - key: skip-in-detached-child-unreportable
    rounds: [1]
    status: resolved
  - key: open-risks-carry-no-risk
    rounds: [1]
    status: resolved
  - key: prime-push-holds-pair-teardown-lock
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: child-cwd-inside-task-worktree
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: command-trust-bound-not-in-command-producer
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: destruction-chokepoint-constraint-misplaces-orchestration
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: trylock-protocol-reach-into-board-sync-unstated
    rounds: [2]
    status: open
    class: decision
    severity: LOW
    fix: design-wrong
  - key: local-mark-bound-is-premise-not-bound
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: no-row-failure-halts-vs-setting-error
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: hidden-slug-flag-unbounded
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
---

Round 1's 18 findings are all resolved: none reappears in round 2, and the round 2 review confirms the fixes for the lock location, the late child's check, the local-remove guards and batten's command-key layer.
Round 2 raised 8 new keys, none recurring.
The three MEDIUM ones are the prime-side pushes taking the pair's teardown lock, the detached child's cwd and log sink inside the task worktree, and the command trust bound living in batten's lookup instead of the `Command` producer.
The five LOW ones are the destruction-chokepoint wording, the try-lock reach into the board sync, the local-mark bound, the "no row's failure halts the run" wording and the hidden slug flag's bound.

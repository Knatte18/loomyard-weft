---
round: 3
ledger:
  - key: walk-misses-strict-load-routes
    rounds: [1, 2]
    status: resolved
  - key: board-rewrite-assumes-restore-and-commit
    rounds: [1, 2]
    status: resolved
  - key: shed-walk-test-reaches-tmux-in-tier1
    rounds: [2]
    status: resolved
  - key: walk-runs-on-verbs-loading-no-config
    rounds: [1]
    status: resolved
  - key: hubreconcile-doc-start-verb-wording-stale
    rounds: [1]
    status: resolved
  - key: config-layers-background-keeps-unknown-key-rule
    rounds: [1]
    status: resolved
  - key: classification-test-called-unchanged
    rounds: [1]
    status: resolved
  - key: move-parse-failure-branch-unreached
    rounds: [1]
    status: resolved
  - key: seed-child-way-forward-checked-on-authored-text
    rounds: [1]
    status: resolved
  - key: unreadable-board-row-reaches-missing-config
    rounds: [1]
    status: resolved
  - key: profiles-write-rule-contradiction
    rounds: [1]
    status: resolved
---

Round 3 raised no finding and resolved the keys round 2 left open: the dobby's run notices route to a strict load,
the board carry's failure handling, and the shed test that reached tmux.

The round was incremental.
Its review names defects in unchanged plan text that it did not raise as findings, so they have no ledger key;
the round 3 verdict file lists them.

---
round: 2
ledger:
  - key: walk-misses-strict-load-routes
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: board-rewrite-assumes-restore-and-commit
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: shed-walk-test-reaches-tmux-in-tier1
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
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

Round 2 resolved every round 1 key except the two that were BLOCKING then, each now narrowed to MEDIUM.
The walk still misses one route to a strict load, the dobby's run notices (card 9).
The board carry still leaves `board.json` changed when the render fails after the save (card 8).

The one new key is card 9's planned tier1 shed test, which holds no run lock and so reaches a tmux server.

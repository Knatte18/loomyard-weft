---
round: 2
ledger:
  - key: edit-directive-cards-miss-handbuilt-stencil-tests
    rounds: [1]
    status: resolved
  - key: pair-call-unnormalized-worktree-path
    rounds: [1]
    status: resolved
  - key: headcontains-test-repeats-stencilsource-rows
    rounds: [1]
    status: resolved
  - key: card8-new-loomcli-test-file-duplicate-row
    rounds: [1]
    status: resolved
  - key: configengine-scratchdir-bundled-in-hubreconcile
    rounds: [1]
    status: resolved
  - key: decodeleafvalues-unused-path-param
    rounds: [1]
    status: resolved
  - key: walk-writes-config-into-incomplete-pair
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
  - key: batten-stamp-before-seed-assertion-unobservable
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
---

Round 1's six keys were not raised again in round 2 and the review's focus verdicts clear the two MEDIUM ones, so all are resolved.
Round 2 raised two new keys.
F1 (MEDIUM design): the reconcile walk writes config into a pair `Topology.Add` has not finished wiring, so it should skip incomplete pairs.
F2 (LOW consistency): card 10's stamp-before-seed test row cannot observe its ordering.

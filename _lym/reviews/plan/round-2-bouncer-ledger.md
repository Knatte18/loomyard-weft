---
round: 2
ledger:
  - key: seed-push-bypasses-destruction-gate
    rounds: [1]
    status: resolved
  - key: hardcoded-branch-names-vs-code-conventions
    rounds: [1]
    status: resolved
  - key: gitrepo-test-rows-spell-side-token
    rounds: [1]
    status: resolved
  - key: interrupted-clone-reconcile-pushes-unseeded-board
    rounds: [1]
    status: resolved
  - key: seeded-board-has-no-upstream
    rounds: [1]
    status: resolved
  - key: abandon-seeded-clone-no-reaching-test
    rounds: [1]
    status: resolved
  - key: never-pushed-row-passes-without-namespace
    rounds: [1]
    status: resolved
  - key: card9-misses-namespace-sites
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: deadcode-guard-producer-before-consumer
    rounds: [1]
    status: resolved
  - key: join-fetch-skips-remote-refs-lock
    rounds: [1]
    status: resolved
  - key: join-reset-reads-origin-head-for-records
    rounds: [1]
    status: resolved
  - key: transient-refusal-way-forward-unreached
    rounds: [1]
    status: resolved
  - key: card4-edit-targets-unchanged
    rounds: [1]
    status: resolved
  - key: card1-tests-name-no-tier-file
    rounds: [1]
    status: resolved
  - key: seed-bullet-unreachable-reason
    rounds: [1]
    status: resolved
---

Round 1 raised fifteen findings; round 2 resolved fourteen of them.
The one that remains is `card9-misses-namespace-sites`, raised again as F16:
one-repository sites `probeLiveLocalWeft`, `readSideOwnerArchives`, `Fabric.PushWeft` and `targetWeftTip`'s fetch
still reach no planned row, and the overview's claim that card 6's Checkout row reaches `targetWeftTip` holds only for the branch-absent exit.
The fix is local: correct the overview's decision text and add four bullets and Uses lines to card 7's second-machine test.

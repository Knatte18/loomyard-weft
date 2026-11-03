---
round: 2
ledger:
  - key: two-root-helpers-no-split-root-test
    rounds: [1]
    status: resolved
  - key: verbs-fixture-plan-has-no-root
    rounds: [1]
    status: resolved
  - key: disksizes-worktree-root-base-unpinned
    rounds: [2]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
---

Round 1 raised two test-coverage gaps in the subpath-anchor change.
Round 2 confirms both are closed: split-root tests cover the two-root helpers, and the verbs fixture plan now has a `root:` with targets committed only under the anchor directory.
Round 2 raises one new finding, F1, mapped to `disksizes-worktree-root-base-unpinned`: no test pins the worktree-root base of `batcher.DiskSizes`, because the verbs fixture uses the identity batcher.
It is `scope`, so it is not gating.

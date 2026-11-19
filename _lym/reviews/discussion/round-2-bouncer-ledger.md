---
round: 2
ledger:
  - key: d3-seed-push-no-place-before-cloneandwire
    rounds: [1]
    status: resolved
  - key: d3-anchor-refusal-catches-two-repo-weft
    rounds: [1]
    status: resolved
  - key: d4-full-name-delete-reports-missing-ref-deleted
    rounds: [1]
    status: resolved
  - key: board-branch-name-in-namespace-undecided
    rounds: [1]
    status: resolved
  - key: d3-force-replace-bound-checked-too-early
    rounds: [1]
    status: resolved
  - key: d3-rerun-adopts-leftover-not-replaces
    rounds: [1]
    status: resolved
  - key: open-risks-researchable-board-body
    rounds: [1]
    status: resolved
  - key: d2-gitrepo-param-list-misses-upstream-and-deletes
    rounds: [1]
    status: resolved
  - key: d1-git-facts-unpinned
    rounds: [1]
    status: resolved
  - key: d3-teardown-covers-refused-push-only
    rounds: [1]
    status: resolved
  - key: d9-pattern-backgrounds-list-incomplete
    rounds: [1]
    status: resolved
  - key: d8-omits-single-repo-flag
    rounds: [1]
    status: resolved
  - key: plan-writer-note-files-wrong-package
    rounds: [1]
    status: resolved
  - key: board-branch-fixed-main-breaks-fabric-readers
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
---

Round 1's thirteen findings are all resolved by the round-2 review's prior list.
The fix for the undecided board branch name fixed it as `main`, which produced the one round-2 finding:
fabric derives the prime's records branch from the board's checked-out branch, and the clone builds the board on the code store's branch.
So a code default branch other than `main` breaks the orphan sweep, the `--single-repo` bind and a second machine's clone.
The fix changes a decision in D1 and its acceptance criteria, so it needs a fresh review.

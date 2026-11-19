---
round: 3
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
    status: resolved
---

Round 1 raised thirteen findings, all resolved by round 2.
Round 2's fix fixed the board branch as `main`, which raised one BLOCKING finding:
fabric derived the prime's records branch from the board's checked-out branch.
Round 3 found that D1 and D3 now name the board branch as the constant `main` and record the prime's records branch in `lym.primeRecordsBranch`.
Round 3 reports no findings, so no key remains open.

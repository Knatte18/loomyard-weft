---
round: 1
entries:
  - key: dirty-worktree-row-vacuous-merge-base
    status: closed
    finding: F1
    reason: "ef9ddf931 commits other/other.go before the --all run in the dirty-worktree row, so the whole-green commit differs from f.mergeBase; the later commit, the edit and the wantDerivation check are unchanged"
---

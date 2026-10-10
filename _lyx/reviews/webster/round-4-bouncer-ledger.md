---
round: 4
ledger:
  - key: prebuild-go-build-uncapped
    rounds: [1, 2]
    status: resolved
  - key: copyhub-dirty-lyx-warp-unrecorded
    rounds: [1]
    status: resolved
  - key: tmuxkit-nested-kit-dir-socket-overflow
    rounds: [1]
    status: resolved
  - key: repo-toplevel-duplicates-lyxcwd
    rounds: [1]
    status: resolved
  - key: harness-prebuildlyx-untested
    rounds: [1, 2]
    status: resolved
  - key: gate-tagged-build-failure-untested
    rounds: [1]
    status: resolved
  - key: resumearming-test-asserts-absence
    rounds: [1]
    status: resolved
  - key: tmuxkit-tests-missing-parallel-comment
    rounds: [1]
    status: resolved
  - key: tmuxkit-package-comment-wording
    rounds: [1]
    status: resolved
  - key: file-headers-missing
    rounds: [2]
    status: resolved
  - key: tmuxkit-sweep-leaves-pane-processes
    rounds: [3, 4]
    status: open
    class: design
    severity: MEDIUM
  - key: lyxcwd-root-not-symlink-resolved
    rounds: [3, 4]
    status: open
    class: design
    severity: MEDIUM
---

Rounds 1 and 2 findings do not recur and stay resolved.
Both round-3 design keys recur in round 4 at MEDIUM.
`tmuxkit-sweep-leaves-pane-processes` (F1): the kit's `sweep` does not reap pane process trees, and no test covers a pane that ignores SIGHUP.
`lyxcwd-root-not-symlink-resolved` (F2): `gitWorktreeRoot` resolves the found root but not the start, so a symlinked start diverges from `git rev-parse`.

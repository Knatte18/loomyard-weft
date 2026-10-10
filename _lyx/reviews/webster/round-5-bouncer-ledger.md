---
round: 5
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
    status: resolved
  - key: lyxcwd-root-not-symlink-resolved
    rounds: [3, 4]
    status: resolved
---

Round 5 raised no findings.
The two keys open in round 4 are resolved: the focus questions on `sweep`/`afterRun` and on `gitWorktreeRoot` were settled against HEAD as not defects.
Every earlier key stays resolved.

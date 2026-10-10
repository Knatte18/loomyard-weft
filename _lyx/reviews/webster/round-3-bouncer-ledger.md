---
round: 3
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
    rounds: [3]
    status: open
    class: design
    severity: BLOCKING
  - key: lyxcwd-root-not-symlink-resolved
    rounds: [3]
    status: open
    class: design
    severity: MEDIUM
---

Rounds 1 and 2 findings (prebuild `-p` cap, harness prebuild test, file headers, and the round-1 leftovers) do not recur in round 3 and are resolved.
Round 3 adds two design findings.
F1 (BLOCKING): the tmuxkit sweep does not guarantee the swept servers' pane processes are gone, so `reedcli` failed Publish verify on a leftover `sleep 300` under load.
F2 (MEDIUM): `lyxcwd` resolves the worktree root lexically where `git rev-parse` resolved symlinks, so a symlinked hub gets a different reed socket key.

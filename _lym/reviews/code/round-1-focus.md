---
round: 1
focus:
  - "internal/fabricengine/destroy.go `ignoresEveryPath` and `removeIgnoredTaskPath`: the files are listed relative to the task path, but check-ignore runs in `l.WorktreePath()`. Settle whether the ownership proof holds for paths that sit outside that repository, for example a `.gitignore` rule anchored to a directory, a nested ignore file under the leftover, or a path that is tracked in the checked-out repository. Also settle whether the `-z` output parse and the exit-1 handling are correct."
  - "internal/fabricengine/destroy.go `filesUnderTaskPath` and `removeIgnoredTaskPath`: settle whether symlinks, empty directories, a non-directory at the path, and a `.git` entry at any depth are each handled safely before the delete, and whether the delete still goes through `removeWorktreeLeftover` with containment and the `KindPathRemoved` record."
  - "internal/fabricengine/add.go at the `ErrTaskPathTaken` check: settle whether the new branch changes any earlier refusal order, whether the failure paths (stat, list, check-ignore, delete) return the right error, and whether rollback and the recorder behave when the delete has run but a later step of Add fails."
  - "pattern/PATTERN-fabric-boundary.md and internal/fabricengine/doc.go: settle whether the third exception in the destructive-primitive rule matches what the code does, and whether the Add doc comment and the package doc agree with the code."
  - "internal/fabricengine/add_taskpathleftover_integration_test.go `TestAdd_TaskPathLeftover`: settle which refusal cases of `removeIgnoredTaskPath` have no coverage (non-ignored file, registered worktree, `.git` entry, check-ignore failure, failed delete), and whether the test duplicates an existing `ErrTaskPathTaken` test."
  - "Commit message of 83c066fa against the diff: settle whether each claim (no worktree registers it, no `.git` entry, check-ignore ignores every file) is borne out. The decision record holds only the issue text under `## Goal`; settle whether the diff stays inside its expected outcome, a reused slug whose old path holds only build output is not blocked."
---

The decision record carries the board entry alone, so the entry is the measuring stick and the other sections are empty.

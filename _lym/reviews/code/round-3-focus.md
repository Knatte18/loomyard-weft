---
round: 3
focus:
  - "Check `internal/fabricengine` non-test code for any remaining literal spelling of the detached-HEAD marker `(detached)`, including the comparison of `entry.Branch` in `internal/fabricengine/originreconcile.go`, against the `detachedBranch` constant in `internal/fabricengine/worktreelist.go` that `parseWorktreePorcelain` and `worktreeBranchAt` use; settle whether `PATTERN-named-constants` holds."
  - "In `internal/fabricengine/remove.go` (`removePair`, `refuseWhenNothingRemains`) and `internal/fabricengine/pairbranch.go` (`pairTaskBranchLocal`, `inspectPair`), settle whether a pair whose only local remnant is its launcher directory, with bare-slug branches on origin and a non-empty `branch_prefix` now set, is named from origin so Remove with remote deletes the branches origin holds."
  - "In `internal/fabricengine/pairbranch.go` the records step of `PairTaskBranch` skips a records worktree whose repository root cannot be resolved; settle whether a test in `internal/fabricengine` keeps the records worktree path present while `RecordsRepoRoot` fails and pins that `PairTaskBranch` and `PairParts` finish without error."
---

Focus entries name where to look; the reviewer still reads the whole range.

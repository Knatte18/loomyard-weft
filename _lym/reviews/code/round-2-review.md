---
verdict: APPROVED
---

## Verdict

No findings. The diff makes the change the amended entry asks for: after git removes a worktree, `fabric remove` deletes a directory still on disk on both sides of the pair, and Add's rollback does the same for the weft worktree.

The range reviewed is `7f0f65b3..HEAD`.
`lym gate test ./internal/fabricengine` at tier 0 and tier 1 reported its targets as carried (0 discovered, 0 failed, exit 0), so I did not re-run the tests myself.
I judged them by reading.

## Focus questions

- **Add rollback with a refilled weft directory.**
  `rollbackAdd` now passes `t.leftover` into `removeWeftWorktree` (`add.go:852`).
  `TestAddRollback_FinishesRefilledWeftWorktree` forces a failed Add with a launcher blocker, refills the weft directory through the refill seam, and asserts three things: Add returned an error, the refill ran, and the directory is gone.
  It would fail without the seam being threaded through, because the refilled directory would stay.
  Verdict: covered.
- **Dropping the registration guard.**
  `finishGitWorktreeRemoval` returns `gitErr` when `List` shows the target as a linked worktree.
  The "locked sibling" row locks the sibling with `git worktree lock`, so git refuses and the worktree stays registered.
  The row asserts the error holds "weft worktree remains", the directory and its `.git` are intact, and no `KindPathRemoved` was recorded.
  Without the guard, `pathPresent` would hand the directory to `removeWorktreeLeftover` and delete it, so the row fails.
  The `gitErr` plus failed-finish path has its own sibling row, "a sibling worktree git deregistered whose removal fails is returned and kept".
  Verdict: covered.
- **Failed listing and the failure message.**
  A `List` error returns `gitErr` before any recursive delete (`listErr != nil || listsLinkedWorktree(...)`), so a failed listing cannot license deleting a worktree git refused to remove.
  The clean-exit failure message tells the operator to stop whatever refills the directory, delete it by hand, and run remove again for a pair that remains; one test row pins that text.
  Verdict: not a defect.

## Other checks

- `finishGitWorktreeRemoval` lives in `destroy.go`, so `PATTERN-fabric-git` holds.
  Hub writes go through `removeWorktreeLeftover`, which removes under the hub container.
- The doc comments are standalone, and the touched `doc.go` paragraph matches the new behavior.
- `SetLeftoverSeamForTest` now sets the two fields instead of replacing the struct, so it does not clear a refill set earlier.
- The commit messages carry no `Pattern-Departure:` trailer, and none is needed.
- No stencil, spec or `PATTERN.md` change is declared or made.

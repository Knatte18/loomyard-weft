```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens, covering what the diff's changes force outside the lines they touch.

**List, per change and the set found**

1. `removeWeftWorktree` and `removeLocalWeft` gain a `leftover leftoverSeam` parameter.
   - Callers found: `add.go:852` (rollback), `remove.go:539` (`removeSibling`), `remove.go:339` (Remove's local path).
   - All three are updated, and `topology.go:22-23` holds the seam field. Covered.
   - Rollback is the only caller that passes `force=true`, but a rollback hits the new directory deletion only after a gated git removal that already accepted discarding the target.
   - No other non-test caller exists. Covered.
2. `finishGitWorktreeRemoval` is new, with a clean-exit branch and a git-failure branch.
   - Its callers are `removeWarpWorktreeDir` (`remove.go:1020`), `removeWeftWorktree` (`weftwiring.go:210`) and `removeLocalWeft` (`remove.go:469`).
   - `removeWarpWorktreeDir` keeps its own inline nonzero-exit handling (`remove.go:1041-1054`), which duplicates the helper's logic. That is the focus's "should it have routed through the helper" question, covered below as a finding.
3. `leftoverSeam` gains a `refill` field.
   - `SetLeftoverSeamForTest` now assigns fields, so it no longer wipes a refill that was set earlier. Its existing callers pass only `remove` and `sleep`: `destructivegaps_integration_test.go:416,440`, `remove_guard_integration_test.go:71`, and the new test at line 111. Covered.
   - `export_test.go:118` builds `leftoverSeam{}` for a direct `removeWarpWorktreeDir` call. Covered.
4. The package `doc.go` paragraph at 586-591 is updated.
   - `doc.go:937` still says `removeWorktreeLeftover` "takes ownership and dirtiness from the gated `git worktree remove` its caller just ran".
   - `PATTERN-fabric-git.md:43` says the same. Both stay true, because the clean-exit path also follows a gated run. Covered.
   - `doc.go:613` says `removeLocalWeft` "removes the sibling worktree through `removeGitWorktree`". It now also finishes through `finishGitWorktreeRemoval`. This is a small omission and is not worth a finding.
5. The new `finishGitWorktreeRemoval` sits in `destroy.go`, which `PATTERN-fabric-git` makes the only file allowed a destructive primitive.
   - It calls `removeWorktreeLeftover` and `removeContainedPath` only through the destroy chokepoint.
   - `remove.go` and `weftwiring.go` call the helper and no `os.Remove*`. Covered.
   - No PATTERN change is declared, so none is owed.
6. The commit has no `Pattern-Departure:` trailer, and none is needed. The commit message says "Remove now checks … on both sides" and matches the diff.
7. No stencils are touched.

**Findings**

- LOW / consistency — `internal/fabricengine/remove.go:1041-1054` against `destroy.go:1403-1424`.
  - Summary: `removeWarpWorktreeDir` keeps its own hand-rolled copy of the "deregistered and left on disk" finish. The new `finishGitWorktreeRemoval` implements the same decision, and the sibling side uses it.
  - Divergence: the two copies word their errors differently. The task-side copy says "it is not a linked worktree of this repo and finishing its removal failed", and the helper says "deregistered it and failed".
  - Consequence: the next change to the ownership rule must land in two places. The helper's doc comment says its non-clean-exit outcome is the same check.
  - Evidence: `remove.go:1041-1053`, `destroy.go:1409-1421`.
  - The summary and commit message disclose the split ("Its handling of a nonzero exit is unchanged"), so it is a design choice. The same drift risk is behind it, though.
  - A routing change would alter the task-side error wording that existing tests pin (`destructivegaps_integration_test.go:359`), which argues for leaving it as is.

No other finding comes out of this lens.

**Tool calls**
1. `grep -rn` over the repository's `.go` and `.md` files for `removeWorktreeLeftover`, `leftoverSeam`, `removeWeftWorktree`, `removeLocalWeft` and "deregistered but left". I wanted callers, docs and PATTERN mentions beyond the files already in context, and the grep returned them.

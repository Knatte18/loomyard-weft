```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals the fixer carried out (F1, F3, F5, plus F2/F4's test shape) through the fix-proposal lens.

**Findings**

- **LOW** — `internal/fabricengine/originreconcile.go:383`, the `BranchRewritten` case. The `lymVerbDir(b.CodeWorktree)` change is sound for the reachable paths.
  - Code targets set `codeWorktree: worktree` (`originreconcile.go:717`).
  - `outcome.CodeWorktree = target.codeWorktree` is assigned before `stopWayForward` runs (`originreconcile.go:890`).
  - A records target can't reach `BranchRewritten`, because `judgeBranch` only yields it for code branches, as the review states.
  - `moves.go` call sites such as `originreconcilemoves.go:399` and `:409` build outcomes through the same path. I did not confirm they set `CodeWorktree`. If one builds a `BranchOutcome` without it, `lymVerbDir("")` returns an empty directory, and the line reads "in " with nothing after it.
  - Before the fix, the code branch used `b.Worktree`, so this is a possible regression on any path that leaves `CodeWorktree` unset.
  - Evidence: `lymVerbDir` returns `""` for an empty root (`failedpush.go:241`), and `originreconcilemoves.go` was not read.

- **LOW** — `internal/fabricengine/pull.go`, `weftDirtyStop`, and `anchorstop.go`, `upstreamAnchorStop` (F1 fix). Anchored coverage is missing for the F1 `upstreamAnchorStop` change and the `pull.go:442` records-conflict push step.
  - The fixer dropped the `pull_integration_test.go` edits because that fixture's warp and weft are not hub siblings. `lymVerbDir` resolves no marker there and falls back to the root.
  - Neither change has an anchored test, so reverting them breaks nothing.
  - The review's F2 fix asked for those rows, so this should go in the fixer-report's deferred section with that reason, not be silently dropped.
  - Evidence: the gate log shows `lym fabric push` in `T/002` with no anchor segment, even with the marker set.

- **no finding** — F5 (doc.go narrowing). The text now names failed-push notices and reconcile stops, which matches what the diff converts.

- **no finding** — F4's invalid-marker row. `../escape` falls back to the root and tier 0 passed.

**Tool calls**
1. `grep -n "CodeWorktree"` over `originreconcile.go` and `originreconcilemoves.go`, to check that the `CodeWorktree` field is set for `BranchRewritten` outcomes.
2. `sed -n` over `originreconcile.go` lines 195–200, plus a grep and sed of lines 880–895, to confirm that `CodeWorktree` is assigned before `stopWayForward` runs.

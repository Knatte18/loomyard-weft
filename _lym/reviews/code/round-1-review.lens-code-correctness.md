```yaml
lens: code-correctness
outcome: reported
```

Task: the code-correctness lens over the diff. The only executable change is the `ResolveWithAnchor` removal, plus one comment claim about `primeMark` order that I checked against the code.

**Findings: none.**

Each function or branch, the case I ran, and the result:

- **`lymcwd.ResolveWithAnchor` removal** (`internal/lymcwd/lymcwd.go`). Case: a case-insensitive search of the whole repo for `resolveWithAnchor`, which also covers the private `resolveWithAnchorCore`. No match remains in code, docs, stencils or tests, so nothing is left calling a removed symbol and the build cannot break on it. Correct.
- **`resolveCore` and `ResolveWorktree`** (unchanged). Case: `ResolveWorktree(root)` goes through `resolveCore(root, false)`, which runs `gitWorktreeRoot`, then `buildLocation` with `applyGate=false`. The removed code did not touch this path. Correct.
- **`buildLocation`** (unchanged). Case: the removed `resolveWithAnchorCore` was one of its callers. The remaining caller is `resolveCore`. Behavior is the same for it. Correct.
- **`gate_test.go` comment edit.** Case: the diff changes only `//` lines in the file header and in the `TestCheckCwdAnchorGate` doc. No test code was touched, so no coverage of a surviving function was dropped. The new text says `buildLocation` is exercised directly and `ResolveWorktree` reaches `applyGate=false` after finding the worktree root. That matches `resolveCore`, which calls `buildLocation(..., applyGate)`. Correct.
- **`primeMark` order versus the new `doc.go` text** (`internal/adoptcli/check.go`). Case: input is `own` plus two other running subjects that sort as `a` then `b`. `texts` starts as `[own.Text]`. The `os.ReadDir` order is lexical by subject, so `a` and `b` follow in that order. Result: `own | a | b`. This matches the corrected claim that the other marks follow the subject's own, sorted by subject. The state is running if any joined record runs, else `own.State`, which matches the second sentence.
- **`primeMark` failure and boundary cases:** an unreadable directory returns `own` unchanged. A record that fails to read is skipped with one Warn. A subject's own directory is skipped. An empty directory returns just `own.Text`. Correct.
- **`render.go` header and `RenderRecoveryPrompt` comment.** Case: a blank `gateTiers` goes through `gateTiersOrNone`, which returns `"none"` for an empty string (lines 181, 280, 609-613). The new comment "A blank gateTiers renders "none"" is correct. The recovery stencil reads `_lym/plan/00-overview.md` (`webster-prefix-recovery.md:26`), so the `plan/00-overview.md` correction is correct.
- **`render.go` blank-`specsDir` claim:** the new header text says "The fill refuses a blank one". I did not read the fill code that does the refusing, so I did not verify this claim.
- **Comment-only edits in `cmd/lym`.**
  - `crosscompile_test.go:28` and `main_integration_test.go:32` call `tiergate.Require(t, 1)`, which matches the "tier1" wording.
  - `main_integration_test.go` seeds its repo with `gitkit.Git(t, cwd, "init")`.
  - `exitcode_test.go:105` asserts the "unknown command" error in a JSON envelope.
  - `destructiveguard_test.go:139` has the `repointLink` row.
  - All match the corrected comments.
- **`hermeticenv_test.go` "tier1" claim:** the near-equivalence of the git-spawning set to the tier1 set is a statement I did not test. Code behavior is not affected either way.
- **`stencilseed.go` and the `configengine.md` signatures:** the `stencilseed.go` edit is comment-only, with no code effect. I did not check the `configengine.md` signatures against the code, which is outside my lens.

Tool calls:
1. One `Bash` call: a case-insensitive grep for `resolveWithAnchor` across the repo, plus `internal/adoptcli/check.go` lines 335-400. My context lacked a search for the lowercase private name and the `primeMark` body.

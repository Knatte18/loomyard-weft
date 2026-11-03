```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals the fixer carried out as proposed (F1, F2, F3, F5, and F4's partial form), looking for a case where a proposal is wrong or breaks something.

**Findings**

1. LOW — F4, `internal/battencli/wire.go` (`createRefusal(..., taskPathStands)` and its caller):
   - When `PairWorktreeStands` is true, `ErrTaskPathTaken` now passes through as a bare error. It is no longer a `*battenshed.PreconditionError`, so the halt carries no way forward.
   - The reviewer's case was a registered worktree that `ResolveWorktree` failed on. The operator was previously told to delete it, which was wrong. They now get Add's raw text with no remedy.
   - Evidence: the `TaskPathTakenByARegisteredWorktree` row has `wantSame` true and `wantContain` nil, so nothing pins a remedy for this branch.
   - The reviewer's own alternative was to reword the `Unmet` text so it makes no "not a worktree" claim.
   - It also breaks the `createRefusal` doc claim that a path-taken refusal is "settled by deleting it". For the pass-through case the doc comment gives no remedy, because there is none.

2. LOW — F4, `wire.go` caller:
   - `fabricengine.PairWorktreeStands(location, slug)` is now evaluated as an argument on every create error, whatever its type.
   - It spawns `git worktree list` even for `ErrSourceDirty`, `ErrBranchExists` and the other errors that never read the flag.
   - This is an error-path cost only. It could be avoided by calling `PairWorktreeStands` only inside the `ErrTaskPathTaken` branch.

3. LOW — F3 and F4, an interaction:
   - F3 makes `PairWorktreeStands` return true whenever the path is present and the repo cannot be listed.
   - With F4, an unlistable repo plus a plain leftover directory now passes `ErrTaskPathTaken` through raw. The operator gets neither the "delete it" remedy nor a hint that git listing failed.
   - The unlistable-repo case has no test, which is consistent with F3 not being a coverage finding.

I found no case where F1, F2 or F5 is wrong or breaks a sibling:
- F1: with `refilledKept` not set, `StrayPath` would be empty, so the new test fails.
- F2: breaking the `liveWarpSlugs`, hidden-name, weft-suffix or reserved-name skip makes the third pass report `CodeWorktree` equal to a checked path.
- F2 caveat: the weft-suffix case is caught only through the `notes-weft` path, and the live pair's own `live-pair-weft` directory is not asserted.
- F5: reverting the seam to `os.Stat` makes the plain directory count as a pair, so the `runJSON(t, 0, ...)` calls would exit 1.

Tool calls: none; my context already held every changed file.

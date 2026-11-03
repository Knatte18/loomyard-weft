```yaml
lens: fix-proposal
outcome: reported
```

**Task:** challenge the reviewer's proposals for F1, F2 and F3, all of which the fixer applied as proposed.

**Findings:** no findings.

- **F2:** I built the case that would make the proposal wrong, a batten-supported recipe that lands code without a `Finalize` row. It does not hold. `contracts/recipes/{tapestry,darn,sampler,pawl}-recipe.yaml` each declare a `Finalize` row (`pawl-recipe.yaml:79`, `engine: Finalize`), and only `motif-recipe.yaml` has none (a single match, in a comment). So `ChildLanded` reading false does not skip a real landing for any of them.
  - The seam resolves the task location the same way `ParentBranch` did, so an absent worktree still ends in Stuck as before.
  - Teardown keeps the draft entry of a motif run, because `settleClaim` returns `claimKeep` for a non-run status. The fixer's new `TestSettleClaim_DraftEntryOfFinalizeLessRun` pins this. `Landed: true` in the teardown request is left unchanged, and the review did not ask to change it.
- **F1:** the table now runs once per entry point, with a distinct slug per entry. Replacing the `wire()` call with `loomCommitStatusDeps` would fail the `Wire_FinalizeLessRun…` row, because `FinishedStatus` would stay empty. The tier-1 log shows all four rows passing.
- **F3:** the `LockedIndexFailsTheCommit` row takes the git dir from `fabricengine.RecordsWorktree(location)`, the same worktree `CommitAnchoredPaths` commits in. It passed in the tier-1 log, so it fails for the intended reason. The lock is removed in `t.Cleanup`.

**Tool calls:** 1
- Grepped `contracts/recipes/*.yaml` for `Finalize`, to check that every batten-supported recipe other than motif has a `Finalize` row; my context lacked the recipe files.

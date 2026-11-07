```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fix-consistency lens over the round-1 change set (the `dotgit` test rows, the `gitkit` `parseGitVersion` extraction and test, and the `fabricengine` comment).

No findings.

- **Symlinked-admin-entry row:** it can fail if the symlink resolution breaks. `FindRoot` does no `EvalSymlinks`, so the common dir stays `root/deep/link/.git`. Without resolution, the relative `gitdir` would give `root/deep/rel`, and the row wants `root/rel`.
- **`parseGitVersion`:** the only caller is `SkipBelowGitVersion`, and the extraction keeps its previous parsing. The added `TrimSpace` is harmless.
- **`query_test.go`:** it follows the sibling test files' conventions (file header, `tiergate.Require(t, 0)`, `package gitkit`). `gitkit`'s closed import set is untouched, since the test imports only `tiergate`.
- **`commit_integration_test.go` comment:** the reworded comment matches where `List` now fails. The two other loose `git worktree list` wordings were rejected in the review and are out of scope here.

Tool calls (1 of 6):
- `grep` of `internal/dotgit/dotgit.go` for `EvalSymlinks` and `FindRoot`: my context lacked `FindRoot`'s body, and the symlink row's discriminating power depends on it.

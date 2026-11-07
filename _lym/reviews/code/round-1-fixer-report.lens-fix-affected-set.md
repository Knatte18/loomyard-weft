```yaml
lens: fix-affected-set
outcome: reported
```

Task: confirm the F1, F2 and F3 fixes cover every member of their sets.

- **F1 (ListWorktrees branches):** no findings. The new rows cover each branch the review named: explicit `files` storage, a symbolic ref chain, the six-link depth error, the `ENOTDIR` fall-through to `packed-refs`, a plain file under `worktrees/`, and a relative gitdir through a symlink or with a missing target. The `EISDIR` branch of `readLoose` has no row, but the review's fix did not ask for one.
- **F2 (version parse):** no findings. `parseGitVersion` and `compareDottedVersions` are called from `SkipBelowGitVersion` and the new `query_test.go` only. The three other callers (`websterengine/gitwrap_test.go:59`, `fabricengine/worktreelist_test.go:247`, `dotgit/worktrees_git_test.go:122`) go through `SkipBelowGitVersion` unchanged. `toolfloor` has its own parser and is out of scope, as the review ruled.
- **F3 (stale comment):** one finding.
  - **LOW:** `internal/preflight/preflight_integration_test.go:266`, the comment "Break `git worktree list --porcelain` at the anchor path".
  - **Summary:** the comment may be stale in the same way F3 describes. The review did not name it, and I did not check whether `PrimeName` now reads through `dotgit`.
  - **Evidence:** I found no dotgit use in the `preflight` production files. `PrimeName` lives elsewhere, so I could not confirm the claim.
  - **Not findings:** the other `git worktree list` comments (`hubmutation_integration_test.go:346`, `destructivegaps_integration_test.go:788`, `junction_pattern_integration_test.go:323`) were rejected or left alone by the review as loose wording.

Calls made:
1. `grep -rn "git worktree list"` over test files and `grep` for the version-helper names over `internal`: to rerun the F3 and F2 set searches.
2. `sed -n 258,282p` of `preflight_integration_test.go`, plus a grep of `preflight` production code for dotgit use: to check the one extra F3 site, which my context lacked.

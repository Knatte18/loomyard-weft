```yaml
lens: fix-affected-set
outcome: reported
```

No findings. I checked both findings' sets and the fix covers every member of each.

- **F1:** `showFile` is exposed only through `ShowFileForTest` (`export_test.go:385`), and `TestShowFile_ReadsCommittedBytes` is its only test caller. The three new rows (`ShortSHA`, `AnnotatedTag`, `DirectoryPathIsNil`) sit in that one table. The `v1` tag is created before the table, so the subtests see it.
- **F2:** `resolvedPath` is called twice in `headCommit` (`query.go:46` and `:48`), and `headCommit` is called only from `RevParse` (`query.go:23`). The `SymlinkedWorktree` row passes through `resolvedPath` at both call sites. Without it, the link path would differ from the real worktree path and `ok` would be false, which fails the row. The `verifytree` `headCommit` hits are a different function in a different package.

Further tool calls (2):
1. `grep` for `ShowFileForTest`, `headCommit(` and `resolvedPath` across `internal`. My context lacked the set of callers and test sites.
2. `Read` of `internal/gitkit/query.go` lines 15-75. My context lacked `headCommit` and `resolvedPath`, which the F2 row has to exercise.

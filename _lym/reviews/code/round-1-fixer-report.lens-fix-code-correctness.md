```yaml
lens: fix-code-correctness
outcome: reported
```

no findings

- **F1 rows.**
  - `ShortSHA`: `first[:7]` is 7 hex characters, so `hex.DecodeString` errors on the odd length. Any other non-full length fails the `sha1HexLength`/`sha256HexLength` check. Both go to `ResolveSHA`, then `FileAtRevision`. Without a short-SHA path through `ResolveSHA` the row would error, so it pins behaviour.
  - `AnnotatedTag`: `v1` is not hex of full length, so it goes through `ResolveSHA`. The row fails if the tag is not peeled to a commit.
  - `DirectoryPathIsNil`: the row expects `nil` with no error. It fails if `FileAtRevision` returned a tree listing or an error for a directory.
  - The `git tag -a` call sits before the table and is shared read-only by the parallel subtests, so there is no race.
- **F2 row.**
  - `SymlinkedWorktree`: `headCommit` takes the `FindRoot` result for the symlink path, while `ListWorktrees` yields real paths. The `want` side of the comparison is resolved through `resolvedPath`, and `resolvedPath` resolves each `worktree.Path` too, so the paths match. Remove `resolvedPath` and the paths differ, so `headCommit` returns `ok=false` and the row's `fromFiles=true` assertion fails. That assumes `FindRoot` does not itself resolve symlinks.
  - The `os.Symlink` call has an error check and the link sits under `t.TempDir()`, so it is cleaned up.
- **Callers.** Only test files changed, so no signature or caller is affected.
- **Gate.** The fixer's tier-1 gate on `./internal/gitkit` and `./internal/fabricengine` passed.

Tool calls made:
1. `grep` for `showFile`, `headCommit` and `resolvedPath` definitions: my context lacked the production code.
2. `Read` of `internal/gitkit/query.go` lines 20–90: I needed `headCommit` and `resolvedPath` to trace the symlink row.
3. `Read` of `internal/fabricengine/configchanges.go` lines 100–160: I needed `showFile` to trace the SHA, tag and directory rows.

```yaml
lens: fix-code-correctness
outcome: reported
```

Lens fix-code-correctness: I checked the F1 and F2 changes in `call.go`, `edits.go`, `loomcli/arbiter.go`, `drivewiring.go`, `loomengine/arbiter.go` and `call_test.go`.

no findings

Cases I ran through:
- **`loomcli` `Changes`:** `WorktreeChangedFiles` returns only file paths, never directories, from go-git's status map. `hashFileOrDeleted` therefore never gets a directory, which `os.ReadFile` would reject with a non-`ErrNotExist` error. A tracked file deleted in the worktree hashes as `"deleted"`.
- **Path root:** `filepath.Join(location.WorktreePath(), …)` uses the same root as `worktreeSnapshot`'s `codeRoot`, so the relative paths resolve to the right files.
- **`maps.Equal`:** a nil map and an empty map compare equal, so an attempt with no uncommitted paths still matches.
- **Callers:** `ArbiterAttempt.Changes` and `WorktreeSeam.Changes` are used only in the files above. The `ArbiterAttempt{...}` literals in other packages' tests don't set `Changes`, so the type change breaks none of them.
- **New test rows:**
  - The row that edits an untracked file spawns with `a.go: h1` and leaves `a.go: h2`. The old path-set compare treated that as equal and let the resume through, so the row fails without the fix.
  - The row for an unreadable `Changes` at spawn expects a refusal and no start. With the `err` check dropped, the spawn would go ahead and the row would fail.
  - The "untouched" row keeps equal hashes in a different insertion order and still passes.

Tool calls: 2 reads, 1 to find where `WorktreeChangedFiles` is defined and 1 to read it (`internal/gitrepo/worktree.go`). My context didn't have what that function returns for untracked directories.

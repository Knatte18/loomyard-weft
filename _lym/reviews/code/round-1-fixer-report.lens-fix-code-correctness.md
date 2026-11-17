```yaml
lens: fix-code-correctness
outcome: reported
```

No findings.

- `dirtyOwnedFile` now reads `CodeWorktreeChanges()`, which calls the same `code.WorktreeChangedFiles()` that `Status` used for the warp side. The code-side result is unchanged, and the records side drops out.
- Failure path: a read error is still wrapped as "read the worktree status".
- Callers: `testmap.go:599` is the only caller and the signature is unchanged. Nothing else in `internal/gatecli` calls `fabric.Status()`.
- The new test row fails with the defect in place:
  - Before the fix, `Status` included the weft side, so `caller/records-note.txt` would have matched `caller`'s ownership and `caller` would have been reported unmeasured.
  - After the fix, only code-side paths are read, and the row expects `Built: ["caller"]`.
  - The records checkout's untracked files are listed by `WorktreeChangedFiles` (staged, modified or untracked), so the file is seen.
  - The fixer reported that tier1 passes for the package.
  - I did not run the new row against the old code.
- The `--tags` removal compiles. The refusal format string still has one `%s` and one argument, and the generic `--tags` refusal at `run.go:361` now catches `--map --tags`. The deleted `cli_test.go` row was the only test of the map-specific message for `--tags`.
- Cleanup in the new row: `os.Remove` leaves an empty `caller` directory in the records checkout. Git does not list empty directories, so later subtests are unaffected.

Calls made:
1. `grep` for `WorktreeChangedFiles` and `.Status()` callers: checked that no other `gatecli` code depends on `Status`.
2. `Read` of `internal/gitrepo/worktree.go`: checked that untracked files appear in the changed set, which my context lacked.

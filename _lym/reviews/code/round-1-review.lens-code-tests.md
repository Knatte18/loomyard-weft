```yaml
lens: code-tests
outcome: reported
```

Task: lens code-tests — for each behaviour the diff adds or changes, name the test that pins it.

**Per-behaviour list**

- The diff adds no behaviour and changes none. It is three file deletions under `tools/boltrename` (`main.go`, `rename_test.go`, `testmain_test.go`), 414 lines removed and none added.
- The deleted program's behaviours (renaming `_board` to `_bolt`, rewriting worktree links, process records and workspace paths) no longer exist. A deleted behaviour needs no test, and its own tests went with it, so no orphaned test is left covering removed code.
- Nothing outside `tools/boltrename` references the package or its tests. `git grep -i boltrename` returned no hits in tracked files. `tools/` has no `go.work` or module list that names it.
- No new test, assertion or test-economy question arises, since the diff adds none.

**Findings:** none.

**Tool calls:** none. My inherited context already held the diff stat, the `git grep` results and the file listing.

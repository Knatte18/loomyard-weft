```yaml
lens: fix-affected-set
outcome: reported
```

Task: the fix-affected-set lens, rerunning each set-premise finding's search against the committed fixes.

**Findings**

1. **LOW**, `internal/fabricengine/addanchor.go:1-3` (file header). It still says a new pair's weft is "moved to the requested anchor under a move record", and `Add` removes that record. That is no longer true for the one-repository path: F2's fix moves the records on disk through `moveForkedRecordsToAnchor`, with no move record. The header should name both routes. Evidence: `add.go:696-704` takes the one-repository branch and writes no record, and `add.go:795` removes the record only when `!oneRepo`.

**Searches that found nothing**

- **F7:** `HoldRemoteRefs()` has one non-test call site, `originreconcile.go:860`, and the fix covers it. The only other caller is `withRemoteRefs`, which takes a single lock.
- **F12:** the owed-push fields are set only at `pairteardown/runread.go:237-239`, and `Judge` now clears all four of them.
- **F3:** nothing else still says a `protected` push stays owed.
  - `PATTERN.md:33` and `internal/fabricengine/doc.go:1590` already say `protected` is not owed.
  - The other "stays owed" hits (`PATTERN-daemon-wakeups.md:94`, `dobbyengine/doc.go:128`, `creelengine/doc.go:317`) are unrelated.
- **F2:** `stageNewMarker` and `moveForkedWeftToAnchor` have no callers other than `addanchor.go` and `add.go`.

**Tool calls (3, in the parallel batch plus one follow-up)**

- `grep HoldRemoteRefs()` over non-test Go: my context lacked every call site, which F7's fix had to cover.
- `grep PushOwedWorktree=|PushOwedError=` over non-test Go: my context lacked the other sites that set or clear the owed-push fields, which F12 needed.
- `grep` for "stays owed" and "protected" near "owed" in `*.md` and `doc.go`: my context lacked the other docs that could repeat the stale F3 claim.
- `grep stageNewMarker|moveForkedWeftToAnchor|moveForkedRecordsToAnchor`: my context lacked the full caller set for F2.

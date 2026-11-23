```yaml
lens: fix-affected-set
outcome: reported
```

Task: check that the fixes cover every member of the sets the findings name (F1 callers and nil-cancel sites, F2 prefix users, F8, doc/comment siblings).

**LOW** — `internal/reedcli/doc.go:7`: the package doc still says "Each cycle it reads the OOM kill count of every strand pane's cgroup."
- F1 moved the read out of the discovery cycle into each session's own `watchPaneOOMKills` goroutine, so "each cycle" is stale.
- The sibling docs the fix did update are `watchdog.go:199`, `watchdog.go:318` and `oomnotice.go:1`.

**LOW** — two comments name the old behaviour:
- `internal/reedcli/watchdog_integration_test.go:455` says "a watchdog:off worktree enters the known set but starts no watcher".
- `internal/reedcli/entersession_integration_test.go:20` says "starts no watcher".
- A `watchdog: off` worktree now starts its OOM watch goroutine and only the name-repair loop is skipped.
- Both tests assert `repairsNames`, so the comments should say "no name-repair loop".

Checked with no gap:
- **F1 callers:** the only `enterSession(` callers are `watchdog.go:599` and the two integration tests, and all pass `timing`.
- **F1 nil-cancel sites:** no `cancel != nil` check remains in `reedcli`, and no use of the removed `location` or `oomKills` fields remains outside `oomnotice.go`.
- **F2:** no `paneScopeUnitPrefix` reference remains. The renamed `PaneScopeUnitPrefix` is used in `panememory.go` and `oomnotice.go`.
- **F8:** the busctl move in `capPaneMemoryLocked` was the only spawn the review named; I did not search for other lifecycle spawns logged at Debug.

Calls made:
1. `grep` for `paneScopeUnitPrefix`, `ws.location`, `.oomKills`, nil-cancel and "no watcher/goroutine" phrases across `internal` and `docs`, plus `enterSession(` callers: to cover the F1 and F2 sets.
2. `grep` for `watchdog: off` and `OOM` wording in non-test files: to find stale docs after F1.

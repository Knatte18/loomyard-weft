```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

- **F4:** `refuseUncontainedPath` now matches nowhere in the tree (outside `.lym`, `_lym` and `.git`), so no third citation was left.
- **F6:** `hubRel` has one declaration, in `hubworkspace.go:437`. The callers in `teardownlock.go`, `hubworkspace_write.go`, `hubworkspace.go` and `portals.go` are all in the same package, so the move breaks none of them.
- **F3, F5:** not set-premise findings, so no search was needed.

Tool calls:
1. One `grep` for `refuseUncontainedPath` across the repo, to check F4 for citations the review did not name.
2. One `grep` for `func hubRel` and `hubRel(` callers, to check the F6 move. Context held only the review's list of callers, not the full set.

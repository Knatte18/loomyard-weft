```yaml
lens: fix-affected-set
outcome: reported
```

Lens fix-affected-set: no findings.

- F5: the only non-test `project-` literals are `boardProjectPrefix` at `internal/fabricengine/replay.go:31` and `projectFilePrefix` at `internal/boardengine/render.go:143`. The fix puts a comment on both.
- F1: all callers of `isBoardOverview` and `isBoardPath` are `replay.go:536`, `:638` and `:756`, and they all go through the same predicate. The new unit test and the integration row cover the predicate and the by-entry replay path.
- Left unchecked: F3 and F4. They are single golden rows in one test, not sets. F2 is a `decision`-class finding that went to the parent.

Tool calls:
1. `grep` for `project-` literals in non-test Go files, to rerun F5's search.
2. `grep` for callers of `isBoardOverview`, `isBoardPath` and `boardOverviewFile`, to rerun F1's search.

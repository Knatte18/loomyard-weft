```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

- **F1:** `StoreDecisionRecord` has one production caller, `internal/loomcli/discussionrecord.go:30`. It reaches `Board.StoreDecisionRecord`, the method the new assertions pin, so the fix covers the whole set.
- **F2:** `withMissingRecordsUndiscussed` has one use, in `RenderToDisk` (`render.go:40`). The new assertion in `TestRenderToDiskLinksOnlyAStoredDecisionRecord` pins its copy-only contract directly. The other `RenderToDisk` callers (`ownerset.go:251`, `board.go:197`, `replay.go:471`) get the same behaviour through that one function.

Findings left unchecked: none.

Tool calls:
1. grep for `withMissingRecordsUndiscussed` and `RenderToDisk(`, to enumerate F2's sites; my context lacked a repo-wide search.
2. grep for non-test `StoreDecisionRecord(` callers, to enumerate F1's sites; my context lacked a repo-wide search.

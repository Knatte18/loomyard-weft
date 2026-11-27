```yaml
lens: fix-affected-set
outcome: reported
```

Task: fix-affected-set check, rerunning each set-premise finding's search against the fix.

**Unchecked findings:** none. F2, F6 and F7 were checked against the code and searches. F1 and F8 were checked from output already in context. F5 is disputed, so no fix set exists for it. F3 and F4 are not set premises.

**Findings**

1. `LOW`: `internal/webstercli/verbs_test.go` header (F7 fix), "The way-forward tests reach fabricSync through a failing opener instead."
   - The sentence covers more tests than it describes.
   - `TestBracketVerbs_NoRunInProgressWayForward` (`verbs_test.go:1969-1972`) is also a way-forward test, but it builds with `newSkipGitVerbsFixture` and uses no failing opener.
   - The tests the sentence is true of are the fabric-sync ones: `TestBeginBatchCmd_FabricSyncFailureWayForward`, `TestRecordBatchCmd_FabricSyncFailureWayForward` and `TestRecoverBatchCmd_FabricSyncAndReedBootWayForward`.
   - "The fabric-sync way-forward tests" would say that.

**Covered sets**
- F1: `gofmt -l` on both packages printed nothing after the fix.
- F2: both wiring paths now assert all values the finding named (`SkipGit`, `SkipPush`, the strand name).
- F6: no other lost space showed in the lines the branch adds to the two packages. The search was a pattern match, not a full read.
- F8: the other inline pair shape, `driftwiring_integration_test.go:46`, uses the `backend` anchor and is outside the branch's moved tests.

**Tool calls**
- Grep of fixture and opener uses in `webstercli` tests: the context did not list which tests reach `fabricSync`.
- Diff grep for lost spaces: the context's diff predated the fix.
- `verbs_test.go` spans and `sync.go`: the context held only the file's header.

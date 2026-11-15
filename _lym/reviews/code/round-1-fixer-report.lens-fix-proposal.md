```yaml
lens: fix-proposal
outcome: reported
```

Lens fix-proposal, for F1.

**[MEDIUM] The added assertions pass with the `sessionStopWanted` cases removed.**
- Changed text: the `check: assertSessionCounts(t, w.deps, 1, 1)` added to `ContinueOnAClosedPullRequestAtTheGatedHeadIsDoneAndRecordsIt`, and `assertSessionCounts(t, w.deps, 0, 1)` added to `ReviewEndedOnAnOpenPullRequestOnADirtyTreeHaltsNamingTheFiles`, both in `internal/landingshed/prreview_wait_test.go`.
- Evidence: `endSession` (`internal/landingshed/prsession.go:233-240`) calls `stopRecordedSession` whenever `Call` ends with a verdict or a halt, unless the context was cancelled. Both rows end that way, Done and a Stuck halt, with a live strand. So the strand gets exactly one `StopStrand` even if `sessionStopWanted` never returns true. If the early stop in `wake` runs, `endSession` finds the strand not live and stops nothing, which gives the same count of 1.
- The case: delete `rec.ReviewEnded` from `sessionStopWanted`, or make the Continue case always false. Both rows still report the same starts and stops (the closed row 1/1, the open row 0/1), so they pass. The review's own requirement, that removing either case "must then fail a row", is not met.
- The tests you cite as the model assert something different. `MergedStopsTheSessionBeforeTheReReadOfTheTree` asserts `liveAtRead` equal to `[true, false]`, which is the session being stopped before the second tree read. `assertSessionCounts` alone cannot tell that from the stop at the end of the call.
- Proposed fix: add the `liveAtRead` hook to the `setup` of both rows (wrap `DirtyPaths` to record `sessions.isLive("guid-1")`), and assert it is `[true, false]` before and after the second read. The closed Continue row needs `withLiveStrand` as well. Its session comes from `ensureSession` at entry, which records the strand `guid-1`, so the hook works there without extra seeding.

**Proposal checks that held**
- The `withLiveStrand` seed on the open row is needed, because an ended review at entry starts no session. The 0 starts / 1 stop count is consistent with that.
- The `ContinueOnADirtyTree…KeepsWaiting…` rows still pin that the session is not stopped on a dirty tree.

Further tool calls:
1. `grep stopRecordedSession|stopLiveSession` over the non-test files in `internal/landingshed`: to find every stop path, which my context lacked.
2. `sed -n 214,240p prsession.go`: to read `endSession`, whose body my context lacked.

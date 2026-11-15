```yaml
lens: fix-code-correctness
outcome: reported
```

Findings for lens fix-code-correctness:

**[MEDIUM]** `internal/landingshed/prreview_wait_test.go`: the new `assertSessionCounts` checks pass with the defect in place.
- **Changed text:** the `check` added to `ContinueOnAClosedPullRequestAtTheGatedHeadIsDoneAndRecordsIt`, asserting `assertSessionCounts(t, w.deps, 1, 1)`. The `check` added to `ReviewEndedOnAnOpenPullRequestOnADirtyTreeHaltsNamingTheFiles`, asserting `(0, 1)`, has the same flaw.
- **Evidence:**
  - `Call` runs `p.endSession(ctx, err)` after every `decide` (`prreview.go:83`).
  - `endSession` calls `stopRecordedSession`, which calls `stopLiveSession`, whenever the context is not cancelled (`prsession.go:234-240`).
  - Both rows end with a verdict or a halt, not a cancel.
  - So the live session is stopped once at the end of the call even if `sessionStopWanted` never stops it.
  - Delete the `rec.ReviewEnded` case, or make the Continue case always false, and the wake leaves the session running. `endSession` then stops it, and the counts stay `(1,1)` and `(0,1)`.
  - Neither row fails, so the review's requirement is unmet: "Removing the ReviewEnded case, or the clean-and-gated Continue case, from sessionStopWanted must then fail a row."
- **Fix:** use the `liveAtRead` check that `MergedStopsTheSessionBeforeTheReReadOfTheTree` uses. In `setup`, wrap `w.deps.PullRequest.DirtyPaths` to record `sessionsOf(w.deps).isLive("guid-1")` at each read.
  - The Continue row (record seeded with `withLiveStrand`, with `live["guid-1"]` set in `setup`) should see `[true, false]`: live at the wake's first read, stopped before the re-read.
  - The `ReviewEnded` row should see the same sequence.
  - Without the stop case the second read would still be `true`, so the row fails.

Calls made beyond the context: 3 tool calls.
- `grep stopRecordedSession|stopLiveSession` in `internal/landingshed/*.go`: my context did not show who else stops the session.
- Read `prsession.go:214-253`: to see what `endSession` does.
- Read `prreview.go:68-87`: to confirm `Call` runs `endSession` after every `decide`.

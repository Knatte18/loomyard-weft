```yaml
lens: fix-proposal
outcome: reported
```

**Fix-proposal lens: no findings.**

- **F1** (`dobbydeps_integration_test.go:75`): a dobby built without the seam now fails the "want every seam filled" error. The extra `||` term breaks nothing else, because the check sits before the `Quiet` and `MarkerPath` assertions in the same loop.
- **F2**, row `ARunReadLeavesTheEndedAgentRunsUnread`: with the `len(runs) == 0` guard removed, the seam would be called, return `endErr`, and suppress the notice, so the row would fail. With the guard in place, a run read leaves the seam unread and the 45m10s quiet notice fires, as in the existing quiet row.
- **F2**, row `ANilEndedAgentRunSeamFallsBackToTheNewestHistoryEntry`: with the nil guard removed, the nil seam would panic in `noticeCondition`, so the row would fail. Its expected 45m10s, from history at 30m and a 75m advance, matches the existing history-fallback row. The `newestEnd` set to 74m would suppress the notice if the seam were wrongly read.
- **F3** (`TestDobbyRunNotices_NewestAgentEndReadsOnlyTheTaskWorktreesEndedRuns`):
  - A closure that read the prime location would find a nonexistent root and return the zero time, so the assertion fails.
  - A closure that scanned sibling worktrees would return +2h, not `taskEnded`, so the assertion fails.
  - A closure that counted the later-modified `running` record would return +1h, so the assertion fails.
  - The records omit `eventsPath` and carry `pid: 1`. `ReadNewestRunEnd` only reads the outcome and the file's modification time, and the gate passed.

Tool calls: none; my context already held every changed file and the gate result.

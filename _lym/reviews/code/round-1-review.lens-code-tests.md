```yaml
lens: code-tests
outcome: reported
```

Task: lens code-tests. For each behaviour the diff adds or changes, the test that pins it, or none.

**Per behaviour**
- `ReadNewestRunEnd` takes the newest record mtime among non-running runs: pinned by `TestReadNewestRunEnd/TheNewestEndedRecordWins` (`activity_test.go`). The outcomes in the rows are `done`, `died` and `stopped`.
- `ReadNewestRunEnd` ignores running records: pinned by `OnlyARunningRunReadsZero` and `ANewerRunningRecordIsNoEnd`.
- `ReadNewestRunEnd` skips a directory with no record and an undecodable record: pinned by `ADirectoryWithNoRecordOrAnUndecodableRecordIsSkipped`.
- `ReadNewestRunEnd` returns zero for no runs and for an absent root: pinned by `NoRunReadsZero` and `AnAbsentRootReadsZero`.
- `ReadNewestRunEnd` skips a run whose `run.json` vanishes between the read and the stat: none. A race like this is hard to test, so I raise it only as a NIT.
- `noticeCondition` reads `NewestAgentEnd` only when `Agents` reads no run: none. No row asserts the seam is uncalled when runs exist, or that its error stays harmless then. `judgeAgents` ignores `newestEnd` when runs exist, so only the call or the error would show it.
- `noticeCondition` treats a nil `NewestAgentEnd` as before: none. The harness always sets the seam, so a nil-deref regression would go unseen.
- `noticeCondition` sends nothing on a `NewestAgentEnd` error: pinned by `AnUnreadableEndedAgentRunSendsNothing`. Ignoring the error would raise a quiet notice at 2h.
- `noticeCondition` clears the failure episode when the seam reads again: none. This is a trivial recover path.
- `judgeAgents` with a just-ended run stays quiet: pinned by `NoRunReadCountsAnAgentRunThatHasJustEnded`.
- `judgeAgents` with a stale end still notices, and the text reads the idle from the end: pinned by `NoRunReadFallsBackToTheNewestEndedAgentRun`.
- `judgeAgents` takes the latest of since, history and end, with history newer than end: none. The only new rows have the end newer than history. The new loop could drop `NewestHistory` and still pass these rows, if an existing history-only row did not cover it.
- A `running` run keeps its existing path: no new row needed, since the branch is untouched and the existing running-run rows cover it.
- The `NewestAgentEnd` wiring in `dobbynotice.go` returns an error for an absent task worktree: pinned by the new assertion in `TestWire_MarkerAndReportSeamsRefuseAnAbsentTaskWorktree`.
- `readNewestAgentEnd` reads the row's task worktree runs through the shuttle config: none. Only the absent-worktree error is asserted. Nothing shows the success path scoped to that worktree's anchor, rather than the prime's.
- Duplication: none found. No new test repeats `TestReadAgentActivity` or an existing notice row.

**Findings**
1. LOW, design: `internal/dobbyengine/runnotice_test.go` (the `TestRunNotice_Conditions` table). Nothing pins that `NewestAgentEnd` is not read when `Agents` reads runs, or that a nil seam is handled as before. The code is at `runnotice.go:269-277`. Fix: add a row with a run read and `endErr` set that still behaves as the run path says, and one with the seam nil.
2. LOW, design: `internal/dobbyengine/runnotice_test.go`, the same table. No row has the newest history entry later than the newest end, so the max across the three clocks is only half pinned (`runnotice.go:293-297`). Fix: add a row with `NewestHistory` later than `newestEnd` and the expected idle taken from the history.
3. LOW, scope: `internal/battencli/wire_seams_test.go`. The `NewestAgentEnd` success path through `readNewestAgentEnd` (`wire.go:673-680`) has no test. Fix: add a row where the task worktree holds an ended run record and the seam returns its mtime, and a sibling worktree's run is not counted.
4. NIT, scope: `internal/shuttleengine/activity_test.go`. The stat-race skip in `ReadNewestRunEnd` is uncovered. No fix required unless a seam for it is cheap.

**Tool calls:** none. Everything came from the inherited context.

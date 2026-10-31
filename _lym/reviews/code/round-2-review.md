---
verdict: APPROVED
---

No findings.

Each of the five focus questions is settled against the code and its tests.

- **Unreadable landed-commit file** (`internal/battencli/dobbyreads.go`).
  The read closure in `DobbyRuns` logs any `shedrun.ReadLandedCommit` error at Warn and returns nil, so the run lists as landed with an empty commit and the loop continues to the other runs.
  `TestDobbyRuns_ReadsTheLandOutcomeAndTheLandingCommit` (`internal/beamcli/dobbypairanchor_test.go`) puts a directory at the landed-commit path of a fourth run and asserts that run reads `{pastLand, landed, no commit}` while the ahead, landed and unlanded runs read as before.
  Not a defect.
- **Both `recordLanding` sites** (`internal/landingshed/finalize.go`).
  The first-try site is pinned by `TestFinalize_RecordsTheLandingCommitOnTheFirstTry`, which asserts the order `merge,mark,record landed1,push`.
  The retry site is pinned by the two landing rows of `TestFinalize_MergeInRequiredRetry`, which assert `...,merge,mark,record landed1,push`.
  Deleting either call fails a test.
  `TestFinalize_RecordLandingFailureKeepsDone` pins that a failed head read or record keeps Done and still pushes, and `TestFinalize_StuckReasons` pins that a stuck landing records nothing.
  Not a defect.
- **`landNotices`** (`internal/dobbyengine/landnotice.go`).
  The `Requested` refresh while the entry is on the board is pinned by the rows `PerTaskFollowsARequestWithdrawnBetweenTicks` and `PerTaskFollowsARequestRaisedBetweenTicks`.
  The kept reading after a failed board read is pinned by `AFailedBoardReadKeepsTheFirstReading`.
  Dropping the refresh, or overwriting the reading on a failed read, fails one of these rows.
  Not a defect.
- **`notify_on_land` in `internal/boardengine`.**
  `TestEntryDates` has the row "notify_on_land revises nothing", `TestFabricReplay` has `NotifyOnLandSetByTheCommitLandsOnTheEntry` over the `replayFields` entry, and `TestRunLock` has "notify_on_land change" for the run-held entry.
  Not a defect.
- **Wirings.**
  `TestLandingDeps_RecordLandingWritesWhereTheDobbyReads` (`internal/loomcli/landingdeps_integration_test.go`) writes through the `RecordLanding` seam and reads back with `shedrun.ReadLandedCommit` on the prime, the function `battencli.DobbyRuns` calls.
  `TestHubDobbyDeps_ClearNotifyOnLandClearsTheHubBoardEntry` (`internal/beamcli/dobbydeps_integration_test.go`) exercises the clear wiring over a hub board.
  Not a defect.

The rest of the range was read against the board entry and the decision record, and no scope gap or out-of-scope change turned up.
That covers the past-Land test in `battenrecipe` and its hibernate call site, the `land_notice` key and its refusal, the `NotifyOnLand` field and the note refusal, the view and overview lines, and the nil-legal entry in `envkit`.
The trimmed `lym board upsert` help text is the writer's stated response to the command's byte cap, recorded in `_lym/landing/summary.md`, and the claims of that summary hold against the diff.
No `Pattern-Departure:` trailer is present in the range, and none is needed.

## Changes

Each finding is fixed in its own commit. The tier0 gate for the touched packages and the tier1 gate for `beamcli`, `loomcli`, `landingshed`, `boardengine` and `battencli` pass.

- **F1** (3c774a81, fixed as proposed): the landed-commit read in `DobbyRuns` (`internal/battencli/dobbyreads.go`) logs a Warn with slug and error when `shedrun.ReadLandedCommit` fails, and leaves `LandingCommit` empty. The run lists as landed with an unknown commit and the other runs are driven. The `DobbyRuns` doc comment says the commit is empty when none is readable. `TestDobbyRuns_ReadsTheLandOutcomeAndTheLandingCommit` gains an `unreadable` run whose landed-commit path is a directory.
- **F2** (95c19451, fixed with my own change): the proposed row in `TestFinalize_MergeInRequiredRetry` could not express a first-try landing. That table's merger always returns the merge-in-required error first. I added `TestFinalize_RecordsTheLandingCommitOnTheFirstTry` beside `TestFinalize_RecordLandingFailureKeepsDone`, which the fix key also allowed. It asserts the order `merge,mark,record landed1,push`.
- **F3** (1d572e60, fixed as proposed): `TestLandNotices` gains `requestedByTick` and `failBoardOnTick` fields and three rows. Per-task follows a request withdrawn between ticks, per-task follows a request raised between ticks, and a failed board read keeps the first reading. The failed-read tick also fails in `startReady`, which reads the board after the notice step, so the test expects that error for that tick.
- **F4** (122200e7, fixed with my own change): the revised-date assertion is a step in `TestEntryDates`, where the revised-date behaviour is already pinned, not in `TestNotifyOnLand`. The step upserts `notify_on_land` and expects the revised date unchanged. The test's doc comment now names the field. `TestFabricReplay` gains the row `NotifyOnLandSetByTheCommitLandsOnTheEntry`, with an `asNotifyOnLand` helper.
- **F5** (51dcfaf8, fixed as proposed): `TestLandingDeps_RecordLandingWritesWhereTheDobbyReads` runs the `RecordLanding` seam over a hub and reads the commit back with `shedrun.ReadLandedCommit`. `TestHubDobbyDeps_ClearNotifyOnLandClearsTheHubBoardEntry` runs the `ClearNotifyOnLand` seam over a hub board.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-affected-set: no findings.
- fix-proposal, LOW, F1: disputed, `every fix worse`.

No fork returned without a report.

## Disputed

- **Fork finding, fix-proposal, LOW, on F1 (`every fix worse`).** The finding says a permanently unreadable landed-commit file logs the same Warn on every tick.
  - The premise holds, and the fork itself calls it a trade-off, not a defect. The sibling Warns in `dobbyParentRuns` (`internal/battencli/dobbyreads.go:79`, `:90`, `:98`) repeat on every tick the same way.
  - The only change that removes the repeats is a per-slug episode memory in `DobbyRuns`, a stateless reader. That is a new mechanism for one log line, and the review's own fix for F1 asks for a plain Warn.
  - The other half of the finding, that a transient read error settles the notice as "landed, commit unknown", is the loss D6 accepts and the review's focus asks for.

## Deferred

Nothing deferred.

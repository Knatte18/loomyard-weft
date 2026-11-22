# Bound the history reads behind lym report

`lym report convergence` read memory in proportion to the board's whole history.
Run over a four-week window it grew by about 2 GB every 15 seconds to 9.5 GB resident and froze the machine.
The landing joins walked every board commit with no window, and `tasksAt` kept each parsed `board.json` for the whole report.
`boardDoneAfter` also re-walked the board commits once per time-join run.
A sweep of `internal/reportengine` found the same pattern in the batten source:
`newBattenIndex` walked the whole records branch,
`commitsOf` read and cached a slug's status file at every commit in its history,
and `ComputeTiming` walked the records branch twice, once for landed runs and once for runs in flight.

The bound limits memory only.
The report answers what it answered before, including a landing that no longer depends on which other runs widened a walk.

## Board history (`boardhistory.go`, `landings.go`)

- `readBoardHistory` reads `board.json` once per board commit, oldest first.
  The read starts at the parent walk's start or the oldest time-join window's opening, whichever is older.
  The board in effect at that floor is also read: the first parent of the oldest commit walked, or the tip when no commit falls inside the window.
- It keeps the commit times plus, for each slug, the commit spans where its entry reads done and the spans where it keeps one set of labels.
  No parsed board outlives the read of its own commit.
- `doneAfter` and `labelsAt` answer the time join and the landing kinds from those spans with the same semantics as before:
  the first commit after the window's opening that reads done, and the newest commit at or before the landing that holds the entry.
- A landing's slug that the board in effect at the floor lacks gets its labels from `readOlderLabels`.
  It reads older board commits newest first, a page at a time, until the entry is found or the history ends, and keeps only the labels.
- `joinTime` is split: `timeJoinOwners` computes the windows before the board is read, so their openings can set the floor.

## Batten source (`batten.go`)

- The records walk starts one week (`firstBattenLookback`) before the report's window.
  When a run's fresh seed may lie before the walk's floor, `widen` doubles the lookback and walks again.
  The widening stops at the slug's previous tag, or a week before the run's seed stamp when there is no previous tag.
  A run with no seed to find costs at most one more walk, over the whole history.
- `snapshotsOfRun` reads the status file through a per-call `battenStatusReader`.
  It reads once at each commit of the run's span and at the commit before its seed.
  For the oldest commit walked, that is its first parent, so whether a commit is a fresh seed is still decided exactly at the walk's floor.
- `Compute` and `ComputeTiming` build one index each and pass it to `selectRuns`; `selectInFlight` uses the same index.
- Landing checkpoints come from `readCheckpoints`, a walk of their own that no run's seed changes.
  It opens at the parent walk's start, or at the opening of the oldest pair a tag found no task commit for when that is older.
  Widening the batten walk for another run therefore leaves every landing join as it was.

## Design decisions

- Rows keep the batten snapshots of their own span, because the wake classification reads every snapshot.
  These are the run's own source, not a cache of every revision.
- The package doc in `doc.go` states the walk windows and the one-read-per-commit bound.

## Tests

- `TestCompute_ReadsEachBoardCommitOnceFromTheWindowOn`: each board commit from the floor on is read exactly once, plus the board in effect at the floor, and no older commit is read.
- `TestLandingJoiner_HoldsNoParsedBoard`: no type that `landingJoiner` reaches holds a `boardengine.Task`.
- `TestCompute_ReadsBattenStatusesOnlyOverEachRunsSpan`: a run of a reused slug reads none of the earlier run's statuses.
  A run whose seed lies before the first lookback widens the walk and still reads each status of its span once.
- `TestComputeTiming_WalksTheRecordsBranchOnce`: landed and in-flight runs share one records walk.

## To check by hand

- Run `lym report convergence --since 2026-10-20 --format markdown` from the prime against the real board and watch resident memory.
  No run has measured it against the 9.5 GB case.
- Compare its landings and kinds with the previous binary's for the same window.
- `readBoardHistory`'s older-commit walk in `internal/reportengine/boardhistory.go`: a time-join window opening older than the floor's board is not pinned by a test.
- The history walks in `internal/reportengine/batten.go` and `landings.go` still traverse the whole first-parent line when a run has no seed or a pair has no checkpoint in the window.
- The records walk's doubling in `newBattenIndex` and `widen` is pinned only through the widened-seed case above.
- The joiner test in `internal/reportengine/landings_test.go` that uses reflection to find held boards: check its seam markers and its `TestTiming` keep are complete.

## Merge danger

**Door:** two-way
**Blast radius:** report

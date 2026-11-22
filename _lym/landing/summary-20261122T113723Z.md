# Bound the history reads behind lym report

`lym report convergence` read memory in proportion to the board's whole history.
The landing joins walked every board commit with no window, and `tasksAt` kept each parsed `board.json` for the whole report.
`boardDoneAfter` also re-walked the board commits once per time-join run.
A sweep of `internal/reportengine` found the same pattern in the batten source:
`newBattenIndex` walked the whole records branch,
`commitsOf` read and cached a slug's status file at every commit in its history,
and `ComputeTiming` walked the records branch twice, once for landed runs and once for runs in flight.

## Board history (`boardhistory.go`, `landings.go`)

- `readBoardHistory` reads `board.json` once per board commit, oldest first.
  The read starts at the parent walk's start or the oldest time-join window's opening, whichever is older.
  The board in effect at that floor is also read: the first parent of the oldest commit walked, or the tip when no commit falls inside the window.
- It keeps the commit times plus, for each slug, the commit spans where its entry reads done and the spans where it keeps one set of labels.
  No parsed board outlives the read of its own commit.
- `doneAfter` and `labelsAt` answer the time join and the landing kinds from those spans with the same semantics as before:
  the first commit after the window's opening that reads done, and the newest commit at or before the landing that holds the entry.
- `joinTime` is split: `timeJoinOwners` computes the windows before the board is read, so their openings can set the floor.

## Batten source (`batten.go`)

- The records walk starts one week (`firstBattenLookback`) before the report's window.
  When a run's fresh seed may lie before the walk's floor, `widen` doubles the lookback and walks again.
  If a doubled walk finds no new commit, the next walk covers the whole history, so a run with no seed costs one more walk at most.
- `snapshotsOfRun` reads the status file through a per-call `battenStatusReader`.
  It reads once at each commit of the run's span and at the commit before its seed.
  For the oldest commit walked, that is its first parent, so whether a commit is a fresh seed is still decided exactly at the walk's floor.
- `Compute` and `ComputeTiming` build one index each and pass it to `selectRuns`; `selectInFlight` uses the same index.

## Design decisions

- Landing checkpoints come from the bounded records walk.
  A checkpoint older than the walk opens no time-join window.
  This only matters for a pair that checkpointed more than a week before the window and was not marked done until inside it.
  The package doc states this.
- Labels for a landing come from the board commits read.
  An entry that was absent from the board in effect at the floor and from every commit after it gets no labels, even if an older commit held it.
- Rows keep the batten snapshots of their own span, because the wake classification reads every snapshot.
  These are the run's own source, not a cache of every revision.

## Tests

- `TestCompute_ReadsEachBoardCommitOnceFromTheWindowOn`: each board commit from the floor on is read exactly once, plus the board in effect at the floor, and no older commit is read.
- `TestLandingJoiner_HoldsNoParsedBoard`: no type that `landingJoiner` reaches holds a `boardengine.Task`.
- `TestCompute_ReadsBattenStatusesOnlyOverEachRunsSpan`: a run of a reused slug reads none of the earlier run's statuses.
  A run whose seed lies before the first lookback widens the walk and still reads each status of its span once.
- `TestComputeTiming_WalksTheRecordsBranchOnce`: landed and in-flight runs share one records walk.

## Merge danger

**Door:** two-way
**Blast radius:** report

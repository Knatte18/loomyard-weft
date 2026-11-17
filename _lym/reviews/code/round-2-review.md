---
verdict: APPROVED
prior:
  - id: F1
    status: resolved
    answer: writeKeptCancel calls refuseWhileMergeActive before shedrun.WriteKeptCancel, so a refused commit writes nothing. A finished keep intent naming this machine is selected again on the next call (leftoverpairs.go:66-78) and committed through commitCancelRecord, so a failed commit is retried. TestMarkKeptCancels_CommitsAFinishedKeepIntentThisMachineWroteButDidNotCommit and TestWriteKeptCancel_WritesNothingWhileAMergeIsActive pin both paths.
  - id: F2
    status: resolved
    answer: dueOrphans posts each fresh orphan's notice first and records the first sight only for slugs whose notice posted (orphan.go:235-264). TestOrphans' notice-failure subtest pins it.
  - id: F3
    status: resolved
    answer: RemoveHandedOverPair ends the run's processes, then calls td.EndSession, then td.RemovePair over the same local Request, matching PATTERN-pair-remote's order. TestRemoveHandedOverPair asserts the live session is ended and no origin branch moves. The fixer's reason for keeping RemovePair over Run, which leaves the other machine's batten record alone, holds.
  - id: F4
    status: resolved
    answer: TestTearDownLeftovers_RemovesALandedAndACancelledPairAndTheirWorkspaceFolders asserts the result map, both pairs gone, the cancelled run's batten record gone, the landed entry done and the hub workspace listing neither pair.
  - id: F5
    status: resolved
    answer: TestRemoveOrphanPair pins the archive tag holding the local commit and the uncommitted file, and a refused tag push leaving the pair standing. TestRemoveHandedOverPair pins the local removal.
  - id: F6
    status: resolved
    answer: TestDobbyOrphans_ListsOnlyAPairNoBoardEntryAndNoRunDirectoryNames lists only the pair named by neither, with LastCommit and a ProcessLive that follows a registered process.
  - id: F7
    status: resolved
    answer: TestMarkKeptCancels_MarksOnlyACancelledRunWhosePairStands asserts the returned slug, the committed finished keep intent naming this machine, the pair-gone run left alone and DobbyRuns reading PairKept with KeptSince.
  - id: F8
    status: resolved
    answer: TestDobbyLines gains rows for a kept pair with a Since and with a zero Since.
  - id: F9
    status: resolved
    answer: TestPairPushDeferred covers no flag, a code-side flag and a records-side flag.
  - id: F10
    status: resolved
    answer: TestClearDoneRuns_LeavesARunWhosePairStandsForTheLeftoverTeardown asserts no clear while the pair stands and one clear once it is gone.
  - id: F11
    status: resolved
    answer: A PairLastCommit error skips the worktree through the per-slug unreadableOrphanTip episode set instead of failing the listing.
  - id: F12
    status: resolved
    answer: requireStillOrphan reads the board, the batten run directory and the slug's live processes before anything is archived. TestRemoveOrphanPair has one refusal row for each.
  - id: F13
    status: resolved
    answer: TestPairLastCommit covers either tip being newer and the missing records worktree.
  - id: F14
    status: resolved
    answer: TestDobbyRuns_ReadsTheKeptCancelTheHandOverMarkTheOwnerAndTheBoltHold pins PairKept, KeptSince, HandOverMarked, OwnedElsewhere and BoltHeld.
  - id: F15
    status: resolved
    answer: TestOrphans' failing-reading subtest keeps the last due list and recovers on a later reading.
  - id: F16
    status: resolved
    answer: The kept-pairs bullet now follows the runs bullet's continuation line in internal/dobbyengine/doc.go.
  - id: F17
    status: resolved
    answer: internal/pairteardown/doc.go gains a Leftover pairs section describing RemoveHandedOverPair and RemoveOrphanPair.
  - id: F18
    status: resolved
    answer: The lift paragraph of the lym tender resume help is rewrapped near 80 columns like the surrounding help text.
---

Round 2 is incremental, so it raises a new finding only at BLOCKING on a changed hunk.
No such defect was found, and every round-1 finding is resolved.
The changed packages build (`go build` over battencli, dobbyengine, pairteardown, beamcli, fabricengine and tendercli).

Verdicts on the focus directive's questions:

- `MarkKeptCancels`: a refused commit writes no intent, since the merge check now runs before the write.
  A failed commit leaves a finished keep intent naming this machine, and the next call selects and commits it again.
  A failed commit is therefore retried.
- `dueOrphans`: the first sight is recorded only after its notice posted, so a failed post starts no grace.
- `RemoveHandedOverPair`: the order is owned processes, then reed session, then removal.
- `RemoveOrphanPair`: the board, the batten run directory and live processes are read again before the archive.
- `DobbyOrphans`: a `PairLastCommit` error skips that worktree with one Warn per episode and no longer fails the listing.
- Tests: each behaviour the directive names has a test asserting it.
  The redundancy report lists no contained test, and no new test duplicates another's assertions.
- Docs: `internal/pairteardown/doc.go` describes both removals.
  The kept-pairs bullet no longer splits the runs bullet in `internal/dobbyengine/doc.go`.
  The `lym tender resume` help keeps its width.

## Rejected

- `internal/reedengine/bindings_integration_test.go` widens the recorder waits to 30 seconds, a test-timing change outside D7's scope.
  It is no BLOCKING defect, so this incremental round raises nothing on it.
- The `internal/pairteardown/doc.go` sentence saying `RemoveHandedOverPair` serves a run handed over "from inside its pair" is narrower than the kind it serves.
  The dobby takes any marked hand-over the registry reads as owned elsewhere.
  It is no BLOCKING defect, so this incremental round raises nothing on it.
- `RemoveOrphanPair` does not read the last commit again before the archive, so a commit in the window after the last probe tick can still be torn down.
  The archive tags keep that commit, and the window is at most one probe interval, so this is no BLOCKING defect.

# Fixer-report, round 1

## Changes

- **F1** (fixed as proposed): `startAction` in `internal/fronttask/hub.go` takes the process start as a seam; `startDetached` holds the detach, start, release and the `testing.Testing()` guard.
  `TestStartAction` in the new `internal/fronttask/hub_test.go` asserts the spawned arguments equal `adopt check <subject>`, the directory, the output file, the spawn log and pid, the logged failed start, and the refusal of a key outside the table.
- **F2** (fixed with the review's first option): `fronttask.File` logs a failed record write after the release and returns nil, since the released entry starts and `Settle` records it.
  A `RecordWriteErrorAfterReleaseStillFiles` row in `TestFile` pins it; the `File` doc and the package doc state it.
- **F3** (fixed as proposed): an empty condition-stop report halts with `darnConditionStopReportMissing`, which names the stop marker path.
  A `TestDarnWrite_Call` row pins it.
- **F4** (fixed as proposed): `resumeHaltedChildren` reads `ParentOf` and filters by subject before `ChildStatus`.
  The new `AFailedStatusReadOfAnotherSubjectsChildAddsNoLine` subtest of `TestGreenEnd` pins it.
- **F5** (fixed as proposed): `TestHubSettleDeps_OwnedHereReadsTheOwnerOnTheHubBoard` covers the prime and a beam project, each owned by another machine and by this one.
- **F6** (fixed as proposed): `docs/overview.md` names `internal/fronttask`, the start-ahead past `max_runs` and the closed-table `on_land` action on the owner only.

## Self-check

- fix-consistency, LOW: two comments still said "the stop report alone" (`darnwrite.go` `Call` doc, `loomshed/doc.go`): fixed as proposed.
- fix-code-correctness, LOW: after a failed record write, `Remove` before a `Settle` runs no action: accepted trade-off of F2, see Notes.
- fix-code-correctness, LOW and fix-proposal, LOW: beam branch of the owner read untested: fixed as proposed (beam rows).
- fix-proposal, MEDIUM: overview said the action runs when the entry is removed, though `Remove` drops the record first: fixed, the overview now says removed by hand with `lym board remove`.
- fix-proposal, LOW: `ParentOf` now runs for every non-done task and a failed parent read of another beam's child still adds a line: disputed as unavoidable, see Notes.
- fix-proposal, LOW: `File` returning nil loses the action if the entry lands or is removed before any settle: accepted trade-off, see Notes.
- fix-affected-set: no findings.

## Notes

- F2: a front task whose record write failed has no action until the next dobby settle records it; the window is one tick, and the review's own fix text accepts it.
- F4: the parent must be read to filter by subject, so a failed parent read of another beam's child still adds a line to this subject's notice.

## Disputed

None.

## Deferred

None.

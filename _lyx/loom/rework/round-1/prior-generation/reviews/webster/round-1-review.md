---
verdict: APPROVED
---

No findings.

The range is `git diff $(git merge-base main HEAD)..HEAD`: `_lyx/loom/rework/` does not exist, and `product.parent` in `_lyx/shed/convergence-plan/status.json` is `main`.

Checked against the focus directive:

- **Tolerant readers (cards 1 to 7).**
  The new `ParseSeedTolerant`, `ParseStatusTolerant`, `ParseArbiterAttemptDir` and `haltnote.Parse` take bytes or a directory's files keyed by name.
  They tolerate unknown keys and run none of the live reader's validation.
  The diff leaves the strict readers unchanged (`ParseSeed`, `decodeSeed`).
  The `blobread.go` change is a behavior-preserving extraction (`changedPaths`, `dirTreeAtRevision`).
- **No side tokens (cards 6, 10 to 15).**
  A case-insensitive grep of `internal/reportengine` and `internal/reportcli` for warp, weft, host, `_lyx` and `.lyx` finds nothing.
  Fabric is reached through `ArchiveTagPrefix`, `ArchiveTagSlug` and `TaskCommitOf`.
- **JSON shape (cards 10 to 14).**
  Every tag in `reportengine` is snake_case.
  Absent figures are nil pointers or maps (`WakesPerRun`, `Paused`, `DriveReports`, `FixShare`, `UnmappedShare`, `SegmentWeek.Mean`, `LandIncomplete`), and `ratio` returns nil for a zero denominator.
- **Reroute entries (cards 7, 12).**
  `countHandSteps` counts goto and reroute entries except the indexes in `AccountedChild`.
  `ClassifyRecords` fills that list for the `PR-Gate` reroute that follows a moved-parent resume.
- **Wake mean (card 14).**
  `meanWakeSources` omits `land-ready-tasks`, and `absentCounts` still reports it absent for every run.
- **Git reads and the landing joins (cards 8, 9, 13).**
  `readLandings` takes `ParentTip` and `BoardTip` from the snapshot and reads from git objects.
  `joinExact` runs before `joinTime`, and `joinTime` skips tags whose exact join was attempted.
- **Tests.**
  The tests of the touched and new packages pass under `lyx gate test`, and the test budget file and docs were updated in the same commits.
  I found no duplicated coverage.

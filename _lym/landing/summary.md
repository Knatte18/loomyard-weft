# drift: the judge snapshot exempts lym's usage and judgement records

Every drift judge halted at Drift-Verify or Drift-Recheck with `the judge session changed what it may not`, naming files lym wrote itself.
A resume could not clear it, because each new judge session wrote a new usage record and halted again.
A judge that changed nothing now passes the snapshot check.

## What changed

- `loomRecordsDirs` in `internal/loomcli/driftwiring.go` now holds the anchored `shedrun.UsageDirRel()`, so `_lym/usage/session-<id>.json` is exempt on the records checkout, in a subdirectory-anchored hub too.
- `JudgeSnapshot.Changes` in `internal/driftengine/judgesnapshot.go` now exempts the timestamped siblings of `judgement.yaml`, on both checkouts, as it already did `judgement.yaml` itself.
  The seat producer's stale-output archive renames an earlier judge's file to `judgement-<stamp>.yaml`, or `judgement-<stamp>-<n>.yaml` on a same-second collision, after the snapshot is taken.

## Design decisions

- The new `isJudgementPath` matches only a sibling in the same directory with the same extension whose name carries the archive stamp, read through `shedadapters.ArchivedRunDirSegment`.
  A plain `judgement-` prefix in `loomRecordsDirs` would also have exempted any other file a judge wrote under that name, such as `judgement-notes.yaml`, which stays a named change.
- `ArchivedRunDirSegment` carries a run-directory name but holds the one pattern for the archive stamp, so it is reused for a file name with its extension trimmed rather than copied.

## Bound

An edit the judge session makes under `_lym/usage` or to an archived judgement goes unseen, like one under the other exempt paths.
Neither holds code, the expectation or the drift records.

## Tests

- `TestJudgeSnapshot` gains a row for the archived judgement, with and without the collision suffix, and a row for files beside it that stay named.
- `TestLoomRecordsDirs_HoldTheUsageDirectoryUnderTheAnchor` pins the usage path for a root anchor and a subdirectory anchor.

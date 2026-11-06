# drift: the judge snapshot ignores lym's usage and archived judgements

Every drift judge halted at Drift-Verify or Drift-Recheck with `the judge session changed what it may not`, naming a new `_lym/usage/session-<id>.json` and sometimes a `_lym/drift/judgement-<stamp>.yaml`.
Both files are lym's own bookkeeping, not edits by the judge.
Deleting them and running `lym loom resume` re-ran the judge, which wrote new ones and halted again, so no verb got a run past the halt.
A judge that changes nothing no longer halts.

Closes #73
Closes #72

## What changes

- `loomRecordsDirs` in `internal/loomcli/driftwiring.go` now holds the usage directory (`shedrun.UsageDirRel()`) under the location's anchor, so the records-checkout check leaves out the usage record of every session of the run.
- `JudgeSnapshot.Changes` in `internal/driftengine/judgesnapshot.go` leaves out the timestamped copies of `judgement.yaml` beside it, on both the code and the records checkout.
  The copies are what lym keeps of an earlier judge's file when the next judge starts.
- The `driftengine` doc comments describe both exemptions.

## Design decisions

- The archived-judgement test is `isJudgementPath`.
  A path is exempt only in the judgement file's own directory, with its extension, and with a base name that `shedadapters.ArchivedRunDirSegment` reads as an archive of `judgement`.
  That reuses lym's own archive naming, including the numbered `-1` form, instead of a looser prefix match.
  A neighbour such as `judgement-notes.yaml`, a `.txt` copy, a copy in a subdirectory or a differently named file is still reported as a change.
- The usage directory is exempt as a whole on the records checkout, like the other loom records paths, and is anchored the same way, so it also matches under a subdirectory anchor such as `src/backend/_lym/usage`.
- Stated bound: an edit the judge makes under an exempt path goes unseen.
  The paths are lym's run bookkeeping and hold no code, expectation or judgement record.

## Tests

- `TestJudgeSnapshot` has rows for archived judgements, plain and numbered, on the records checkout and on the code checkout, and a row that still names the look-alike files above.
- `TestLoomRecordsDirs_HoldTheUsageDirectoryUnderTheAnchor` checks the usage directory at the root anchor and at a subdirectory anchor.

## To check by hand

Not run: a live drift judge in a hub, to confirm that a Drift-Verify or Drift-Recheck passes with the usage record and an archived judgement present.
The blocked runs named in the issue (`cuttings-fitted-rheology`, `api-density-coefficients`, `baseline-provenance-outside`) can be resumed with `lym loom resume` once this lands.

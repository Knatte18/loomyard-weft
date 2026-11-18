---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed
  - id: F7
    disposition: fixed
  - id: F8
    disposition: fixed
  - id: F9
    disposition: fixed
  - id: F10
    disposition: fixed
  - id: F11
    disposition: fixed
---

## What changed

- **F1**: `Deps.ReplacedDecisionRecord` now takes the snapshot's records head.
  `appendedDecisionRecord` passes `s.Records.Head`, and the loomcli wiring returns found false unless the newest replace of the record was made after that head.
  A new `fabricengine.RecordsCommitIsAfter` decides it by ancestry.
  An empty head admits any replace.
  `doc.go`, the `Deps` doc and the `Changes` and `appendedDecisionRecord` docs say so.
  `TestJudgeSnapshot` gained the row `DecisionRecordRolledBackToAReplaceBeforeTheSnapshotIsNamed` and asserts the head the seam is asked about.
  The fixture's fake answers by that head, so an engine that ignored it fails the row.
- **F2**: `paths.go` declares `logsDirName` and `findingsFileName`, and `NewPaths` builds `LogsDir` and `Findings` from one `logsDir`.
- **F3**: `groupRuledCells` sorts each group by row before taking its first and last cells.
  The findings table has a row whose judge rules the cells in reverse order.
- **F4**: `TestReplacedDecisionRecord_ReadsTheNewestReplaceOfTheRecordMadeAfterTheSnapshot` (tier 1, in `recordhistory_integration_test.go`) commits real replaces over a forged hub.
  It covers the newest replace winning over a newer replace of another path, an empty head, a replace at the head, a head at the tip, and a branch with no replace of the record.
  `commitRecordVersion` now returns the commit's SHA.
- **F5**: the fixture judge takes a per-cell reason (`reasonOf`) and a reverse-order switch.
  New table rows cover cells that differ only in reason, unkeyed lines, and added cases.
- **F6**: `cappedFindings` reserves the separators and the omission note, so the findings part stays within `haltFindingsLimit`.
  The table asserts the findings part's length, the exact count of kept plus omitted findings, and a cut first finding ending with the ellipsis.
- **F7**: the resume subtest asserts the recorded halt keeps its time and holds only `b.go` after the second call.
  A pass that stops on a capture fault after the restore asserts the halt is cleared and the mark stays open.
- **F8**: `AChangeDatedBeforeTheSessionOpenedSaysSo` sets the file's mtime before the mark's `OpenedAt` and asserts the wording.
- **F9**: `recordGuardHalt` records an unreturned session's changes with an empty `At`.
  `changeTime` treats an empty `At` as no halt end, and the first check after the session returns sets the time.
  `AResumeOverASessionThatNeverReturnedNamesWhatWasRestored` covers it.
- **F10**: the findings table asserts the exact line of a cell the capped reason leaves out.
- **F11**: `TestJudgeSnapshot` has `ChangePath_` subtests that pin `GuardChange.Path` for a code, records, decision-record, scratch, deleted and moved-HEAD change.
  They are a separate table beside the main one, so the main table's rows keep their shape.

Gates: `lym gate test ./internal/driftengine` (tier 0) and `lym gate test --tier 1 ./internal/loomcli ./internal/fabricengine` pass.
`go build` and `go vet` pass for `battencli`, `envkit`, `loomcli` and `driftengine`.

Commits group findings by shared files, not one per finding: F2; F1, F4 and F11; F3 and F5 to F10 together; and a follow-up for the self-check.
The round-edit rule ran all edits before one gate, and the findings share `verify.go`, `verify_test.go` and `judgesnapshot_test.go`.

The comment gate then found a wrapped break in the `RecordsCommitIsAfter` doc and two over-long lines in the doc of `TestReplacedDecisionRecord_ReadsTheNewestReplaceOfTheRecordMadeAfterTheSnapshot`.
Both comments now break at sentence boundaries, one sentence per line, in a gate-repair commit.
The gate then read the test doc's name-only first line as a wrapped break, so its first sentence now sits on one line under 120 characters.

## Self-check

Forks returned: fix-code-correctness, fix-consistency, fix-proposal, fix-affected-set.

- fix-code-correctness, LOW, the F1 engine row passed with the defect in place: fixed (the fixture fake answers by the head it is asked since, so the row fails if the engine ignores the head).
- fix-proposal, LOW, same finding as above: merged into the line above.
- fix-code-correctness, LOW, a first check of a returned session after a never-returned halt words a resume re-check: disputed, premise false (see Disputed).
- fix-proposal, LOW, same finding as above: merged into the line above.
- fix-consistency, LOW, the const block comment in `paths.go` is cosmetic: not raised by the fork itself, no change.
- fix-consistency, LOW, the `changedSince` doc says "after a guard halt": fixed (now "after a recorded guard check").
- fix-affected-set: no findings.

## Disputed

- **Self-check finding, fix-code-correctness and fix-proposal, LOW: lead wording after a never-returned halt.** Ground: premise false.
  The case needs a recorded halt with an empty `At` to survive until the judge returns.
  `settleOpenSession` returns the halt reason before any respawn while changes remain, so the judge is not respawned.
  Once the changes are restored, `recordGuardHalt` clears the halt in that same check: its `len(changes) == 0` branch sets `halt` to nil.
  A returned check therefore never sees an empty-`At` halt unless the records mismatched, where `recordGuardHalt` writes nothing and the guard names that edit itself.
  Evidence: `internal/driftengine/verify.go:632` (`settleOpenSession`), `internal/driftengine/verify.go:766` (`recordGuardHalt`).

## Deferred

Nothing deferred.

## Rulings

No parent ruling reached this fixer.

---
verdict: APPROVED
prior:
  - id: F1
    status: resolved
    answer: appendedDecisionRecord passes s.Records.Head to Deps.ReplacedDecisionRecord, the loomcli wiring returns found false unless RecordsCommitIsAfter holds for the newest replace of the record, and the TestJudgeSnapshot row DecisionRecordRolledBackToAReplaceBeforeTheSnapshotIsNamed pins the rollback as a change.
  - id: F2
    status: resolved
    answer: paths.go declares logsDirName and findingsFileName once, and NewPaths builds Findings from the same logsDir value as LogsDir.
  - id: F3
    status: resolved
    answer: groupRuledCells sorts each group by rowOf before taking first and last, and the GroupEndsFollowTheRowsWhenTheJudgeRulesThemOutOfOrder row asserts the range and both endpoint values under reversed rulings.
  - id: F4
    status: resolved
    answer: TestReplacedDecisionRecord_ReadsTheNewestReplaceOfTheRecordMadeAfterTheSnapshot drives the production seam over real commits, covering another path's newer replace, the after-snapshot restriction and a history with no replace of the record.
  - id: F5
    status: resolved
    answer: Rows pin cells that differ only in reason, an unkeyed line range and added cases as one finding each; a removed case takes the same branch, and its case name alone already keeps it its own group.
  - id: F6
    status: resolved
    answer: cappedFindings counts separators and reserves the omission note within haltFindingsLimit, and the table asserts the findings part's length, the omitted count and the ellipsis cut of an over-long first finding.
  - id: F7
    status: resolved
    answer: The resume subtest asserts the halt keeps its first At with only b.go's change, and that a check finding nothing leaves the mark open with GuardHalt nil.
  - id: F8
    status: resolved
    answer: AChangeDatedBeforeTheSessionOpenedSaysSo asserts the before-the-session-opened wording.
  - id: F9
    status: resolved
    answer: recordGuardHalt records the changes of an unreturned session with an empty At, and AResumeOverASessionThatNeverReturnedNamesWhatWasRestored asserts the resume lead and the restored path.
  - id: F10
    status: resolved
    answer: The table asserts the exact bullet line of a cell the capped reason leaves out, for the repeated key and for the many-cases row.
  - id: F11
    status: resolved
    answer: The ChangePath_ rows of TestJudgeSnapshot assert GuardChange.Path for code, records and scratch changes, and empty for a deletion and a moved HEAD.
---

No new BLOCKING defect sits on a changed hunk, and every prior finding is resolved.

On the focus questions:

- A judge commit that edits or removes an existing entry is still named when the only replace predates the snapshot's records HEAD.
  The seam's newest replace is then no proper descendant of the snapshot head, so it answers found false, and the snapshot copy, which holds the entry, is the only base.
- `NewPaths` declares the logs segment and the findings file name once each, and `Findings` lies under `LogsDir` by construction.
- A group's first and last values come from its lowest and highest rows after the sort.
  The joined reason stays within `haltFindingsLimit`:
  a kept finding is admitted only when the separator and the note for the rest still fit, and a cut first finding plus the note equals the limit.
- After a resume the halt keeps its first `At`, updates `Changes`, and is cleared by a check that finds nothing.
  A session whose judge never returned records its changes with an empty `At`, and the next resume words what was restored.
- The tests pin grouping by reason, unkeyed line ranges, the findings file's lines, `GuardChange.Path` for records and scratch changes, and the production replace-trailer read.
  None duplicates a covering test.

## Rejected

- Assistant F5, carrying prior F5 as LOW for a missing removed-case row: a removed case takes the same branch of `groupRuledCells` as an added case, and each case is one cell keyed by its own case name, so a removed-case row would pin nothing the added-case row and the grouping key do not.

## Notes

- `_lym/landing/summary.md` lags the fixes, though the incremental-round rule admits no finding on it.
  Its Tests section still says `replacedDecisionRecord` has no test of its own, and it lists only two new `TestDriftVerify` cases.
  Its design decision that a check before the judge returns records no halt contradicts `recordGuardHalt`, which now records the changes with an empty `At`.

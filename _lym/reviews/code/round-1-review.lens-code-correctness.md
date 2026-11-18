```yaml
lens: code-correctness
outcome: reported
```

Lens: code-correctness. I walked each function and branch the diff adds or changes with concrete inputs.

**Per function or branch: the case I ran, and whether the result was correct**

- `lockDecisionRecord`: tried an empty `DecisionRecord`, a nil seam, a lock error and a lock success. Correct: the no-op release, the wrapped error and the swallowed release error all match the existing `writeCarryOverLocked` style.
- `TakeJudgeSnapshot` and `Changes` lock span: the lock is held over `Checkouts`, `RecordsChangedSince`, `ReplacedDecisionRecord` and the hashing, then released by `defer`. No caller holds the decision lock while it reaches drift. Correct.
- `appendedDecisionRecord`:
  - A committed add on the snapshot base is exempt. Correct.
  - An uncommitted write is caught early by `Changed`/`Untracked`. Correct.
  - A snapshot copy that is missing falls through to the replace base. Correct.
  - A replace made before the snapshot followed by a committed drop of a later added entry: **wrong**, see finding 1.
- `replacedDecisionRecord` wiring: the path is relative to the anchor (`DiscussionDecisionRecord` is anchor plus rel). The trailer value is written from `deps.recordRel` (decision.go:1409), and `AnchoredFileAt` is called with the same rel. A missing path at the commit gives `found` false. Correct.
- `checkoutChanges` and `scratchChanges`: a deleted path gets no `Path` and an added or changed one gets its absolute path. The sort by `Text` matches the earlier sort. Correct.
- `changedSince` and `recordGuardHalt`, case by case:
  - First return with changes: the halt is recorded at now with its changes, and the lead is the plain text. Correct.
  - Resume with some changes still there: the halt keeps its `At` and updates `Changes`. "restored" is computed against the old `mark` copy, which still holds the earlier changes. Correct.
  - Resume that finds nothing: the halt is cleared, `pass` reads the records again and goes on. Correct.
  - Records mismatch: nothing is written, and the next pass reaches the restore path. Correct.
  - Open session, judge never returned, changes found: halts, but no halt is recorded. **Partial**, see finding 2.
- `changeTime`:
  - A file modified in the same second as the open (fraction dropped) reads inside the window. Correct.
  - A file modified more than 1 s after the halt reads "by another process". Correct.
  - A failed stat returns empty. Acceptable.
- `groupRuledCells`:
  - A single cell is worded as that cell. Correct.
  - Added or removed case kinds are worded one by one. Correct.
  - 150 rows of a repeated key give "n 1 to 150" with the first and last values. Correct.
  - Rows that differ only in reason form separate groups, because the reason is part of the key. Correct.
- `cappedFindings`:
  - An empty set returns "", but no caller passes one. Correct.
  - A single finding over the limit is cut and the cut is made valid UTF-8. Correct.
  - At the cap boundary, the "; " separators are not counted. NIT, see finding 3.
- `writeFindingsFile` and `noMatchReason`: `WorktreeRoot` is wired in production (driftwiring.go:156) and in envkit. A write error is returned after the rulings are already stored, so the next pass rebuilds the halt from the stored rulings. Correct.
- `storeRulings`, `storeCaseRulings`, `passCommand` and the stored path of `checkAndJudge`: errors from building the reason are propagated. Correct.

**Findings**

1. MEDIUM, design, `internal/driftengine/judgesnapshot.go:378-391` (`appendedDecisionRecord`), backed by `internal/loomcli/driftwiring.go:1093-1110`.
   - Summary: the replace base is the newest replace commit of any age, including one made before the snapshot. A committed judge edit that rolls the record back to that old replace text is therefore exempt.
   - Example: the snapshot copy holds R plus an entry E1 added after an earlier replace R. The judge commits R. The snapshot base fails, because E1 is dropped, but the replace base R matches. The removal goes unseen.
   - This contradicts `doc.go`'s claim that "an edit or removal of an existing entry or block are still named as a change". The doc's only stated bound is a commit carrying a replace trailer.
   - Fix: take the replace base only from a replace commit made after the snapshot's records head. For example, require the trailer commit to be among the commits since `s.Records.Head`, or compare it with the `Position` of that head.

2. LOW, design, `internal/driftengine/verify.go:632-636` and `recordGuardHalt`'s `default: return nil`.
   - Summary: a guard halt from `settleOpenSession`, on a session whose judge never returned Done, records nothing. Every later resume therefore repeats the same "the judge session changed what it may not" text, with no "still differs" or "restored" line. The issue asks for that line on a resume.
   - Fix: record the changes found, without a halt time, so a later check can name what was restored.

3. NIT, design, `internal/driftengine/haltfindings.go:204-218`.
   - Summary: the cap counts bytes, not characters, and leaves out the "; " separators, so the joined findings can pass `haltFindingsLimit` by about 2 bytes per finding.

Tool calls:
1. Grepped `WorktreeRoot` in the drift wiring and envkit, the `recordReplacedValue` caller and the `decisionRecordPath` sources. My context lacked whether `worktreeRel` has a root in production, and the path form the replace trailer stores.

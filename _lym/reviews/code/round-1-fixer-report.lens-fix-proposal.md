```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals the fixer carried out as proposed (F1–F11) and build a concrete case where one is wrong.

Proposals checked with no finding: F1, F2, F3, F4, F5, F6, F7, F8, F10 and F11.
- **F1:** the real ancestry check (`RecordsCommitIsAfter`) and the empty-head case both behave correctly.
- **F6:** the cap's reserve arithmetic holds. The note reserved for a kept finding `i` equals the note written when finding `i+1` is dropped, so the text never exceeds the limit.

Findings:

1. **LOW**, `internal/driftengine/verify.go#recordGuardHalt` and `changedSince`, from F9's "empty `At`" proposal.
   - **Summary:** the first guard check after a never-returned session returns still words the lead as a resume re-check.
   - **Case:**
     1. A resume halts over a session that never returned, which records `GuardHalt{At:"", Changes}`.
     2. The judge then returns, and `judgeSessionChanged` calls `changedSince` with `returned` true.
     3. The lead is built from `mark.GuardHalt != nil`, so it reads "the resume re-checked the judge guard … restored since the last check", although this is the first halt of the returned session.
     4. `At` is set to now only in the record, after that lead was built, so `changeTime` still uses the "now" window end and cannot date a later edit as another process's until the next check.
   - **Evidence:** `verify.go` `changedSince` (`if mark.GuardHalt != nil` picks the resume lead) and `recordGuardHalt` (`if at == "" && returned { at = stamp(now) }`). No test covers a never-returned halt followed by a return.
   - **Effect:** wording and dating only; the halt itself is still raised.

2. **LOW**, `internal/driftengine/judgesnapshot_test.go`, new row `DecisionRecordRolledBackToAReplaceBeforeTheSnapshotIsNamed`, from F1's test proposal.
   - **Summary:** the row passes with the original defect in place.
   - **Reason:** the fixture fake `replacedBeforeSnapshot` returns found false itself, which is the filtering the fix moved into `replacedDecisionRecord`.
   - **What does pin the fix:** the `replacedSince` assertion and the loomcli rows (`AReplaceAtTheSinceHeadIsNotAfterIt`, `NoReplaceIsAfterTheTip`).
   - **Caveat:** this is not a flaw in the proposal; it concerns only the engine row's value.

Further tool calls: none; my context already held every changed file.

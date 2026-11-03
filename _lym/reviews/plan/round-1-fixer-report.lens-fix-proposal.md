```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge each review proposal the fixer carried out (F2–F9) by finding a concrete case where the change is wrong or breaks another part of the artifact.

Findings:

1. **MEDIUM**: card 2's F5 bullet ("A failure of that removal is returned as a failed move is") and card 5's new rename-help sentence ("the error … names the record file left at the old slug's name, which the entry's next Discussion approval stores again under the new slug").
   - **Summary:** The F5 proposal puts a second failure under the move's error text, and that text, plus the help built on it, is false for this case.
   - **Case:** Take a flagged entry `a` that has no `decision-a.md`, and rename it to `b` while an orphan `decision-b.md` exists. The orphan cannot be removed, for example because it is a directory. The step then returns card 2's move error, `rename moved the entry but not its decision record: %w: the record stays at decision-a.md until …`. No file is at `decision-a.md`, and the record that was not moved is not missing: an orphan at the target name is left in place, which `DecisionRecord` will serve as `b`'s record. That is the outcome F5 exists to prevent.
   - **Effect:** Card 5's help and skill text then tell the operator that a record is waiting at the old slug, so the operator is pointed at the wrong file.
   - **Fix:** Give the target-cleanup failure its own wrapped text, naming the orphan at `decision-&lt;new&gt;.md` and the way forward (remove it by hand). Card 5's help should then cover both errors.

The other proposals hold up against the cases I built:
- **F2:** An orphan left by a failed removal is overwritten by a later `StoreDecisionRecord`, or removed by F5's cleanup on a rename.
- **F3:** `hasPlan`/`rowWritten` and `approvalRow` parse the same embedded recipe first.
- **F4:** The copy runs only after HEAD is known to hold every entry.
- **F6, F7, F8:** Wording and fixture changes with no breakage.
- **F9:** A `--set` to another refused prefix yields a different error text, so `newLoadFailure` still rolls it back.

Tool calls: none.

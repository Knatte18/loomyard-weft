```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the F1 proposal (plain promote of a discussed, sketch-labelled note with a missing record file now takes raddle).

Finding (LOW), `store.go` `promoteTo` condition `recipe == "" && current.Discussed && (recordHeld || slices.Contains(current.Labels, SketchType))`: the change turns a loud refusal into a silent release of a raddle run that has no decision record to read.
- Case: a Sketch whose record file was lost through a rename or a partial checkout is promoted plainly. It becomes a released raddle task (`wantRecord: false` in the new row `PlainPromoteOfASketchWithAMissingRecordFileTakesRaddle`), and the operator is never told the record is gone.
- This is not a regression, because `--recipe raddle` already admitted this note before the diff, so the proposal only widens that path to the plain promote.
- Whether the raddle run fails cleanly on a missing record is not visible in my context, and I made no tool call to check.
- The alternative the review offers, naming `--recipe raddle` in the refusal, would keep the operator's deliberate choice. It would also leave `lym board promote <slug>` as a dead end for a labelled note.
- I give this LOW, as an open question and not a defect.

No other breakage found.
- With the defect in place the new row fails: recipe "" with the label kept is refused by `checkSketchLabel` (`labels.go:57-67`), and the row wants no error.
- `Discussed` is required, so an undiscussed labelled note is unaffected.
- The label-restore branch at `store.go:790` still needs `recordHeld`, which is right because the label is already present in this case.
- The `promoteTo` doc comment and `doc.go:327` agree with the code.
- `doc.go:322` and `:325` stay true.

Tool calls: none; my context already held every changed file.

```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals for F2–F6 as the fixer applied them, through the fix-proposal lens.

**LOW: F5, `cmd/lym/main.go` registration comment**
- The reworded comment is one line of about 145 characters of text: "A records replay merges heddle.json by entry through heddle's Bolt replayer, which also claims HEDDLE.md and renders it afresh after each replay."
- The prose rule caps a line at 120 characters and says to split at a clause boundary, here the comma before "which".
- The reviewer's wording, applied literally, produced the over-long line. Two `//` lines, split at that comma, would keep it under the cap.
- Neighbouring comments in that block are also long, but the changed line is the one this fix introduced.

**LOW: F2, `internal/heddleengine/overview.go` note format**
- The fix writes `"\nNote: %s.\n"` around `layers.DerivedWarning`. That warning ends with `%v` of the error from `agentconfig.Load`.
- If that error already ends in punctuation, the note gets a doubled "..", for example "...: .." Wrapped YAML or validation errors can also span several lines, which would break the note's markdown paragraph.
- The test only checks the prefix `"\nNote: the effort defaults are left out of .claude/settings.local.json: "`, so it would not catch either case.
- I did not read `agentconfig.Load`'s error texts, so I have not seen a real error that triggers this. Dropping the trailing `.` from the format string would remove the double-punctuation risk.

**LOW: F2 and F4 commit attribution**
- Commit `f2ea30f4` ("fix F2") also contains the F4 overview rows: the fence-widening and non-object-JSON rows in `overview_test.go`.
- The F4 commit `dae79ad6` holds only the `settings_test.go` row, so the per-finding commit discipline is blurred. The commit message does say so.

I checked these and found no breakage:
- **F3 test:** it fails if `Apply` took the commit's HEDDLE.md, because the request holds "local overview" and "stale overview" and the test compares against a fresh render of the merged store.
- **F4 precedence row:** it distinguishes layer-based from order-based precedence, since the repository entry has order 0 and the derived one 200. It would fail if order alone decided.
- **F6:** the blank line is restored, and the bullet text is unchanged.

Further tool calls: none made, because my context held every changed file.

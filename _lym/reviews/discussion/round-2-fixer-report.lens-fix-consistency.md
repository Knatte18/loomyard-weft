```yaml
lens: fix-consistency
outcome: reported
```

Lens: fix-consistency. I checked whether the decision-record changes agree with the sections they touch, and looked for edits whose twin elsewhere was left unchanged.

**MEDIUM — D5 resume paragraph and the D5 acceptance criterion contradict the new mid-fix line.**
- Changed text: the D5 sentence added after "A resume without a live chair is a fresh spawn (D6 Bound).": "A done writer the probe removed is counted finished only when its outputs exist and its Accept passes, and the line the chair is told says the writer was stopped at the interruption and that any fix the chair had asked of it is lost."
- The paragraph two sentences earlier still covers the same seat, "a done writer whose kept pane the probe removed", and says "the chair is told its part stands and the seat no longer answers".
- So D5 now gives two different lines for one case.
- `internal/seatengine/resume.go:23` says the probe removes "a done one kept by its pane". Every finished writer at a resume is therefore probe-removed, so the "part stands" line never applies on its own.
- The "part stands" wording is the F3 premise the review asked to correct, and it was appended to instead of replaced.
- It also conflicts with D9's new bullet "a part whose fix request was outstanding at a resume is still open".
- The D5 criterion repeats both lines: "tells the chair its part stands" and "tells the chair the fix was lost".
- The new sentence also restates the earlier sentence's finished condition ("all its outputs exist and its Accept passes on them").

**LOW — D2's key list omits the new key.**
- Changed text: D2's new sentence introducing `parts_chair_timeout_min`, and the D2 criterion "the new keys, `parts_chair_timeout_min` among them".
- D2 opens "`internal/loomengine/tapestry.yaml` gains, all hub-tunable …:" and its bullet list still names only the six original keys.
- The new key appears only in a later paragraph, so the list of keys `tapestry.yaml` gains is now incomplete.

**LOW — D6's closing line contradicts the new error mappings.**
- Changed text: the D6 list mapping a writer that failed to start or died to an error.
- The unchanged line after the "Today the engine's `join` stops the chair…" sentence still says "So a failed writer or an exhausted gate leaves the run Stuck at `Parts-Plan-Write` (D1)."
- A writer that died or never started now returns an error, not Stuck, so "a failed writer" in that line is too broad.

Tool calls beyond context:
- `git diff` on the decision record, to see the change set as a diff. It printed nothing, so the file is untracked or unchanged from git's view.
- `grep` over the decision record, to see the changed lines' current line numbers and neighbours, which my context held only as edit fragments.

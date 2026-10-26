```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens on the `[-]` skipped / `[=]` halted checklist change. No findings.

Per function or branch, the case I ran:
- **`checklistSkippedRows`** (checklist.go:~885): empty history gives an empty non-nil map. History `[Skipped Review]` gives `{Review:true}`. A non-skipped entry never sets a key. The nil-map read case below shows downstream indexing is safe. Correct.
- **`checklistBox` precedence** (:867-884):
  - Current producer with state Running, Failed, or Paused/Blocked/Awaiting returns its state box before the visit and skip checks. A current producer that also has skip entries still shows its state box. Correct.
  - Visits>0 and skipped gives `[x]`. Visits==0 and skipped gives `[-]`. Neither gives `[ ]`. Correct.
  - Current producer in a state that falls through the switch (for example Done) then follows the visit/skip/not-reached path. Correct.
- **Plain-row call** (:272): `skipped[name]` is read from a map that is never nil. `visits[name]` is the zero value for an unvisited row, so `visits==0` and a skipped row gives `[-]` with no time or count line (the `v.visits > 0` guard). Correct.
- **Skipped row not in the recipe** (:240-244): `ordered` adds only visited rows, so a skipped-only non-recipe row never draws. This is consistent.
- **`checklistReviewBlock`, unvisited branch** (:412-421):
  - Segment with no visit and one row skipped gives `[-]` as a single line. Correct.
  - Segment with some rows skipped and others never reached also gives `[-]` as a single line. This matches the decision record's single-heading-line criterion.
  - Segment with no skip gives `[ ]`. Correct.
  - The loop matches `row.Segment == segment` and ignores `inSegment`. Both select the same rows.
  - A segment whose heading came from a non-recipe name cannot occur, because the review block is only drawn for recipe rows.
- **`checklistReviewBlock`, visited branch** (:422-432): passing `skipped=false` means a skipped row inside a visited segment never boxes the heading. This is intended and commented. A halted current producer in the segment still yields `[=]`. An unvisited row of a visited segment returns `[ ]`, which is ignored, so the heading stays `[x]` as before. Correct.
- **Halted glyph**: `checklistBoxHalted` is now `[=]`, and `checklistBox` returns it on the paused, blocked and awaiting path. No other Go reader of `"[-]"` exists. The grep shows only the new test strings and `doc.go`. Correct.
- **Skip-then-goto-visit** (:272 with visits>0): shows `[x]` with time. `checklistVisitsOf` is unchanged and treats a skip as no visit, so the new test row stands. Correct.

Tool calls:
1. Read checklist.go lines 400-450, because the manifest cut off before the rest of `checklistReviewBlock`'s unvisited and visited branches.

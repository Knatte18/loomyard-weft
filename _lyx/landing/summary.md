# Status overview: drop output paths, time the gate lines

The loom-status overview, the frame `lyx loom checklist` draws in a run's reed session, wrapped nearly every visit line in a normal-width pane, because each line ended with the absolute path of the file its entry wrote.
Gate lines carried no clock time, while visit lines open with one.

## What changes

- `checklistVisitLine` in `internal/loomcli/checklist.go` no longer draws the entry's output.
  This applies to a row's visit list, a review block and the recipe-unreadable list.
- `checklistGateLines` opens each gate line with the UTC `hh:mm` of its gate entry's first span that has a parsable start, as in `10:03 gate lint 10s`.
  A gate entry with no parsable span start opens with `--:--`, the frame's existing mark for an unknown start.
- The package doc in `internal/loomcli/doc.go` and the verb's `Long` help describe the new line formats.

## Design decisions

- The visit still carries its output.
  `checklistReviewBlock` still reads the round number from it to place a review visit in its round, so dropping the display leaves the round grouping unchanged.
- A gate line is timed by its own first span, not by the visit's start.
  That way, two gate entries under one visit show when each one ran.
- Every clock time stays in UTC, as the operator decided, because UTC does not jump with daylight saving time.

## Tests

The `TestRenderChecklist` rows that pin visit and gate lines now expect no output and a leading gate time.
The unparsable-span row now also covers a gate entry whose first span has an unparsable start, which takes its time from the next span, and a gate entry with no parsable start, which reads `--:--`.

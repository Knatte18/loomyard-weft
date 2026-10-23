# Decision record — status-overview-trim

## Goal

Make the loom-status overview, the frame the hidden `lyx loom checklist` verb draws in a run's reed session, readable in a normal-width pane:
no visit line carries the path of the file its entry wrote, and every gate line opens with a clock time as visit lines do.

## Scope

- In: the visit lines `renderChecklist` draws, in a row's visit list, in a review block and in the recipe-unreadable list, drop the entry's output path.
- In: each gate line, one per gate entry under a visit, opens with the UTC `hh:mm` of that gate entry's first span start.
- In: the `doc.go` text, the verb's `Long` help and the tests in `internal/loomcli` that pin the line formats.
- Out: `lyx loom status` and its `--watch` tail, whose state line carries no file path.
- Out: the status file's `output` field, which stays recorded; the review block still reads it to place a review visit in its round.

## Decisions

### Drop the output from the visit line only

Decision: `checklistVisitLine` stops writing the entry's output; the visit still carries the output, and `checklistReviewBlock` still reads its round number from it.
Rationale: the path is what wraps the lines, while the round grouping depends on the value, not on its display.

### A gate line opens with its first span's start

Decision: a gate line reads `<hh:mm> gate <name> <gate time>` and the refusal suffix as before, where `<hh:mm>` is the UTC start of the first span of that gate name that has a parsable start, and `--:--` when none does.
Rationale: the first span start is when that gate entry began, matching a visit line's start time; `--:--` is the frame's existing mark for an unknown start, and UTC is the operator's decision for every clock time in the frame.

## Constraints

- `PATTERN-comment-line-breaks`: every changed Go comment is one sentence per line.
- `PATTERN-test-economy`: the new format is pinned by updating the existing table rows in `checklist_test.go`, adding a row only for the unparsable-start case no row covers.
- `PATTERN-test-speed`: tests stay on the injected `checklistClock`, no wall clock.

## Auto-mode assumptions

- The board entry's "written-file paths" are the visit lines' output values, the only file paths the frame draws.
- A gate entry whose spans repeat under one name opens with its earliest-recorded span, the first in list order.

## Open risks

- An operator who read the round file path from the overview loses it there; the status file and `lyx loom status` still carry it.

## Acceptance criteria

- A visit line with an output draws no output, in a row's list, in a review block and in the recipe-unreadable list.
- A review visit whose output names a round still lands in that round.
- A gate line opens with the UTC `hh:mm` of its gate entry's first span start.
- A gate entry with no parsable span start opens with `--:--`.
- `internal/loomcli/doc.go` and the verb's help describe the new line formats.

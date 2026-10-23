# Status overview: drop output paths, open gate lines with a time

The loom-status overview, the frame `lyx loom checklist` draws in a run's reed session, wrapped every visit line in a normal-width pane because each line ended with the absolute path of the file its entry wrote.
Gate lines also carried no clock time, though visit lines open with one.

- Visit lines no longer end with the entry's output path, in a row's visit list, in a review block and in the recipe-unreadable list.
- Each gate line now reads `<hh:mm> gate <name> <gate time>`, with the refusal suffix as before.

## Design decisions

- Only the display drops the output.
  The visit still carries it, and the review block still reads the round number from it to place a review visit in its round.
  The status file's `output` field is unchanged, and `lyx loom status` and its `--watch` tail are untouched.
- The gate line's time is the UTC `hh:mm` of the first span with a parsable start among the spans of that gate name.
  It matches a visit line's start time, and UTC is kept because it does not jump with daylight saving time.
  When no span of the name has a parsable start, the line opens with `--:--`, the frame's existing mark for an unknown start.
- An operator who read the round file path from the overview loses it there.
  The status file and `lyx loom status` still carry it.

The change is in `internal/loomcli`: `checklist.go`, the pinning tests in `checklist_test.go`, and the `doc.go` text and the verb's `Long` help, which describe the new line formats.

## Not run by hand

The rendering was checked only through the unit tests on the injected clock.
It was not viewed in a live run's loom-status window.

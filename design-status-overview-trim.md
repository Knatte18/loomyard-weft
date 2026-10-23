# Status overview: drop file paths, time the gate lines

`status-overview-trim` · Task · enhancement, loom

## Seen (operator, 2026-10-23)

The overview window of a run's reed session, the `ly:<shortname>:<slug>:loom-status` strand that shows `lyx loom status --watch`, is close to unreadable because it puts too much on one screen.
Each producer line ends with the full absolute path of the file it wrote, such as `.../loom-tail-review/_lyx/reviews/discussion/round-3-review.md`, which wraps every line in a normal-width pane.
Gate lines, such as `gate review 0s` under a producer line, carry no clock time, while producer lines open with one.

## Scope

- Drop the written-file paths from the overview's lines.
- Open each gate line with its clock time, as producer lines do.

The checklist is rendered in `internal/loomcli/checklist.go`.

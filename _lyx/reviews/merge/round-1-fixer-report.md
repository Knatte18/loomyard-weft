# Fixer-report, merge round 1

## Changes

- **F1** (fixed as proposed, second option): `checklistPoll` in `internal/loomcli/checklist.go` calls `reviewWaitingFor(location)` while the state is running and passes the note to `renderChecklist`, which appends ` | waiting <note>` to the state line.
  A hook error drops the note, as `shedverbs.statusWatchLine` does.
  `internal/loomcli/checklist_test.go` gains a row for the appended note, and the `loomcli` package doc names the note on the state line.
  `lyx gate test ./internal/loomcli` is green.

## Disputed

None.

## Deferred

None.

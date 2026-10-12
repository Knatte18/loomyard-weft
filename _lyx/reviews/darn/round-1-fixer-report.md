# Fixer-report, round 1

## Changes

- F1: fixed as proposed.
  `Promote`'s closure in `internal/boardengine/board.go` returns a `promoted` wrapper;
  `boardCriticalSection` unwraps it and passes the slug to `stampDates`, which exempts only that entry's kind change.
  The note-to-task rewrite in `stampDates` is gone.
  `TestEntryDates` gained a note-creation row and a row where an upsert flips the note to a task and `revised` moves.
- F2: fixed as proposed.
  `TestFormatAge` in `internal/boardengine/render_test.go` covers each unit boundary, the unknown date and the future-date clamp.
  The `internal/boardengine` row in `cmd/lyx/testdata/test-budget.yaml` is raised from 77 to 78.

`lyx gate test` is green for `./internal/boardengine` and `./cmd/lyx`.
The `TestFormatAge` rows were appended with a shell heredoc rather than Edit or Write.

## Disputed

None.

## Deferred

None.

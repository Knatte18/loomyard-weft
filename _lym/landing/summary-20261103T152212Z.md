# loom status --watch ignores a wait's growing elapsed time

`lym loom status --watch` printed a new line on every poll while a verify, gate wait, shuttle wait or parent lock wait ran.
The waiting note carries the wait's elapsed time (`verify running 6s (...)`), and the watch deduped on the printed line, so the growing counter read as a change each second.
An agent or Monitor watching a run got one event per second for the length of each gate.

## What changes

- `shedverbs.Hooks.Waiting` now returns a `shedverbs.WaitNote` instead of a string.
  `Text` is the note status shows, elapsed times included; `Steady` is the same note with every elapsed time held fixed.
- The watch tail dedupes on a change key instead of the printed line.
  `statusWatchLine` returns the line and its key, where the key carries the note's `Steady` form in place of its text;
  `PrintStatusLinesOnChange` prints a polled line only when its key differs from that of the line it last printed.
  A row, state, wait kind or wait detail change still prints, and the printed line still shows the elapsed time at that moment.
- loom's hook chain in `internal/loomcli/reviewstatus.go` builds both forms through `timedNote`, which renders each note once with the formatted elapsed time and once with the `<elapsed>` placeholder.
  The gate-wait clauses, holders clause and the chained next note are joined the same way into both forms.
- The one-shot envelope, the human status view and the checklist pane keep using `Text`, so they keep showing the elapsed time.

## Design decisions

- The steady form is a second rendering, not a regex strip of the text: each formatting site knows where its elapsed time sits, so no parser has to guess which tokens are times.
- A holders clause is part of the steady form, so a change of who holds a gate slot prints a line; it is the wait's detail.

## Docs

- The `lym loom status` help, the `--watch` flag text, `contracts/specs/loom-status-spec.md` and `docs/overview.md` state that a growing elapsed time alone is no change.

## Tests

- `TestPrintStatusLinesOnChange_LineSequences` gains a row where only the elapsed time changes between polls and no line prints for it.
- `TestStatusWatchLine_AppendsWaitingNote` asserts the key carries the steady form while the line carries the text.
- `TestShuttleWaiting`, `TestVerifyWaiting` and `TestParentLockWaiting` assert each note's steady form is its text with the elapsed time replaced by the placeholder.

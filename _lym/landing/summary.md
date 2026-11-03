# loom status --watch: stop printing a line per poll while a wait runs

`lym loom status --watch` printed a new line every poll while a verify ran.
The composed activity carried the wait's elapsed seconds (`verify running 0s`, `1s`, `2s`, ...), so the change test saw a new activity on every poll.
That contradicted the verb's own help ("a line only when the composed activity actually CHANGES"), and flooded any agent or Monitor watching a run with one event per second for the length of each gate.

Now the watch prints a line only when the row, the state, or a wait's kind or detail changes.
A wait whose elapsed time alone grows prints nothing new.
The one-shot `status` and the checklist pane still show the elapsed time.

## Design

- The `Waiting` hook in `internal/shedverbs` now returns a `shedverbs.WaitNote` instead of a string.
  `Text` is the note as shown, elapsed time included.
  `Steady` is the same note with every elapsed time replaced by the placeholder `<elapsed>`.
- The wait builders in `internal/loomcli/reviewstatus.go` (gate-slot wait, verify, pause hold, shuttle wait, parent-lock wait) produce both forms from one render function, so the two cannot drift apart.
  Composed notes (gate waits joined with the next hook's note) join both forms in parallel.
- `PrintStatusLinesOnChange` polls a line and a change key, and prints when the key changes.
  `statusWatchLine` builds the key from the status line plus the note's `Steady` form.
  The unavailable line is its own key, as before.
- The one-shot envelope's `waiting` key, the human status view and the checklist pane (`internal/loomcli/checklist.go`) use `Text`, so their output is unchanged.
  `checklistPoll` now takes the waiting hook as a parameter, which lets a test pin that the pane's wait line keeps the elapsed time.
- Updated the `--watch` help text and flag description, `contracts/specs/loom-status-spec.md` and `docs/overview.md` to say the watch ignores elapsed time.

## Check by hand

Not run against a live run: start a verify that lasts several polls, tail it with `lym loom status --watch`, and confirm one line appears when the wait starts, none while the elapsed time grows, and a new line when the wait's kind or detail changes (for example, from waiting for a gate slot to running).

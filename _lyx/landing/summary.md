# reed: intact launch lines, --focus=false never takes the viewer

Parallel `lyx shuttle run` launches into one tmux session could start an agent from a corrupted launch line (`5. '<script>'`), so bash answered `command not found`, Claude never started and the run hung.
Separately, `lyx shuttle run --focus=false` still switched the operator's active window to each new agent window.
Both are fixed in `internal/reedengine`, `internal/shell` and `internal/shuttlecli`.

Closes #627
Closes #628

## What changes

- `lyx shuttle run --focus=false` no longer moves the viewer to the run's own window.
  The new window still carries `@lyx_unseen`, as any unselected agent window does, so the operator can see it opened.
- A fresh pane's launch now types the dialect's discard-line key before the launch payload.
  On POSIX that is `C-u`, so input that reached the booting pane first, such as a viewer's keystrokes or a terminal reply while a window switch landed on it, can no longer prefix the launch line.
  The launch is three separate `send-keys` calls: the key, the literal source statement, then `Enter`.

## Design decisions

- `render.Display` gains `KeepViewer`, because `Focus` false already means "follow the run" for every internal spawner and its meaning must not change.
  `KeepViewer` only suppresses the select of an own-window strand's new window; a set `Focus` still wins, and removing a viewed window still moves its viewer.
  The key is `omitempty`, so older `reed.json` records read as false.
- `lyx shuttle run` sets `KeepViewer` to the negation of `--focus`.
  The flag defaults to true, so `--focus=false` is always an explicit ask to stay put.
  `lyx reed add --focus` is untouched.
- `newAgentWindowSelected` and `newAgentWindowTakesViewerLocked` now take the whole `render.Display`.
  A display with `Focus` or `KeepViewer` is decided without listing the session's windows and panes.
- `shell.Shell` gains `DiscardLineKey`, because which key discards the line is a fact of the dialect, chosen on the same `shell.ForGOOS()` as the launch line.
  POSIX returns `C-u`, which works in the tty's canonical mode and in readline.
  PowerShell returns the empty string: its equivalent, Escape, would merge with the typed `.` into an Alt chord, so pwsh sends nothing extra.
  The key is never the interrupt key, and `internal/shell` stays stdlib-only.
- `internal/reedengine/doc.go`'s focus rules name `KeepViewer`.

## Limits

- Stray input can still reach a booting pane when a `Focus` window or a window taking the viewer by the follow rule is selected; it is discarded only when it arrives before the `C-u`.
- PowerShell panes get no discard key, so the launch-line fix is POSIX-only.
- Out of scope: a flag to open a run's window outside the caller's session, and the `--anchor hidden` startup failure ("provider never became ready").

## To check by hand

Not run: several `lyx shuttle run --focus=false` launches at once from the prime in a live tmux session.
Expect every agent to start, the orch pane to stay the active window, and each new window to be marked unseen.

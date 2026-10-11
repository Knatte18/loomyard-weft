# Batten windows stay in the reed session but get no status-bar button

`lyx batten run <slug> --window` opens a `batten:<slug>` window in the prime's reed session, where the batten process lives.
Each such window showed up as a button on status line 1 of the orch's session, and its output tells the operator nothing useful.
The windows now stay, but are hidden from the bar.

## Change

- `reedengine.OpenWindow` chains a third command onto its `new-window` invocation, `set-option -w -t '=<session>:=<name>' @lyx_hidden 1`, next to the existing `remain-on-exit off`, so the window is marked from the moment it exists.
  `OpenWindow`'s only caller is batten's `--window`, so the marker is unconditional rather than a parameter.
- `status-format[0]` (`statusFormatButtons` in `bar.go`) skips the window button of any window carrying `@lyx_hidden`.
  Strand buttons, VIEW and the session line are unchanged.
- `@lyx_hidden` is display only, like `@lyx_strands`: the status bar reads it and no Go decision does.
- The orch role stencil no longer says the session's window list names every run in flight; it points at `lyx batten status <slug>` for watching a run.

Known edge: the window-button branch still keys on `session_windows > 1`, which counts hidden windows, so a session with one plain window plus hidden ones shows that window's button.
A reed session always holds the strands' window, so in practice any plain window already makes the count exceed one.

## Tests

- `window_test.go` pins the new chained `set-option` in the `new-window` argv.
- `window_integration_test.go` (tmux tag) reads `@lyx_hidden` as `1` on the opened window.
- `bar_integration_test.go` (tmux tag) adds a window carrying `@lyx_hidden` to the fixture and asserts line 1 has no button for it while the plain `extra` window keeps its own.

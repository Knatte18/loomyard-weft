# Reed reuses only its own hidden windows and logs every window it opens

The stray `bash` window seen in the prime reed session has no source in reed's code.
Every `new-window` reed issues names its window with `-n`, which turns `automatic-rename` off,
so no reed window ends up named after its shell.
A fresh boot's `new-session` window does start named `bash`,
but the same operation splits Selvage into it, reaps the boot pane and renames the window `shell`.
The stray pane `%1984` was the last pane created before the overview's `%1985`,
so no Selvage split followed it, which rules out a boot.
The session pins `default-command` to the configured shell (`/usr/bin/bash`),
so a window opened by hand, through `prefix c` or the status bar's right-click "New After", gives exactly the window that was seen.
A hand window at the end of the list is then moved ahead of `shell` by the shell-window check,
which matches its place at index 1, right after the overview.

## What changes

- `OpenWindow`'s same-name lookup (`findWindowLocked` in `internal/reedengine/window.go`) now reads only windows carrying `@lym_hidden`, the marker every window it opens carries.
  Before, it skipped only `@lym_own` windows, so a dead window the operator opened by hand under the same name was killed and replaced.
  Reconcile already ended only `@lym_own` windows, so reed now closes or reuses only windows it opened itself.
- A fresh boot logs its first window at Info as `reed: opened the session's first window`, with window id, pane id and name, read by one `display-message` after the session comes up.
  A failed read logs at Debug and never fails the boot.
- The `reed: opened own window` and `reed: moved a strand pane out of the shell window` lines now carry the window name.
  `reed: opened window` already did.
  With these, every window reed opens leaves an Info line naming its window and pane, so a window without one came from outside reed.

## Design decisions

- The fallback the task names was taken: no code path was found, so the change adds the trace lines rather than a fix at a guessed site.
- The `reedengine` package doc's window-order section records the trace lines, the shell `default-command` that a hand window inherits, and the rule that a hand window stays.

## Tests

`TestOpenWindow` in `internal/reedengine/window_test.go` now marks hidden windows in its listing rows.
A new row pins that a dead, unhidden window of the same name stays and a new window opens.

# reedengine: reuse only hidden windows and log every window opened

A stray idle `bash` window appeared in the prime's reed session at a prime bring-up, bound to no strand, and no log named it.
No path in the code leaves a window with no command.
This change narrows what reed may touch and makes every window it opens traceable.

## What changes

- `OpenWindow` in `internal/reedengine/window.go` now looks up an existing window by name only among windows carrying `@lym_hidden`.
  Before, the lookup skipped only `@lym_own` windows, so a window the operator opened by hand under the same name could be adopted, or killed if its pane was dead.
  Now a hand-opened window of that name stays, and a new hidden window opens beside it.
  `windowListFormat` reads `@lym_hidden` instead of `@lym_own` for this.
- A fresh boot logs one Info line, `reed: opened the session's first window`, naming the window id, pane id and window name.
  It comes from the new `logBootWindowLocked` in `lifecycle.go`; a failed or malformed read logs at Debug and never fails the boot.
- The "opened own window" and "moved a strand pane out of the shell window" Info lines now also carry the window name.
  Together with the existing detached-window line, every window reed opens in the session is named in the trace log.
- `doc.go` records which window opens come from which reed step.
  A window named after its shell outside a fresh boot came from no reed step.
  The session's `default-command` is the configured shell, so a window the operator opens by hand with `prefix c` runs that shell.

## Design decisions

- The stray window's cause is not settled.
  The code review found no `new-window` before a `respawn-pane` that a race or failure could leave behind.
  A hand-opened tmux window fits the evidence, so the fix is the guard on what reed reuses plus the trace lines, as the task allowed.
- Hidden windows are the only ones the name lookup reads, since every window `OpenWindow` opens carries `@lym_hidden`.
  Reed ends only a window carrying `@lym_own` and reuses only one carrying `@lym_hidden`.

## To check by hand

- `logBootWindowLocked` is covered by `TestLogBootWindow_NamesTheFirstWindowAndLogsNoInfoLineOnAFailedRead` against the scripted fake tmux only.
  Confirm on a live tmux server that a fresh prime boot writes the first-window Info line with the expected id, pane and name.
- Read `TestOpenWindow` in `window_test.go`: the live and dead cases now distinguish hidden from unhidden windows, and the old own-window case became the unhidden-window case.

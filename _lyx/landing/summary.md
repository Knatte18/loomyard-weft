# OpenWindow counts a read-back from another window as gone

`OpenWindow` no longer reports a window whose command exited at once as a `remain-on-exit` read-back mismatch.
This removes the flake in `TestOpenWindow_RealTmux/ClosesWhenItsCommandExitsAtOnce`, which failed with `window @3 read "on"`.

The cause is in the read-back after `new-window`.
When the command has already exited, the window is closed by the time `display-message -p -t <window id>` runs.
tmux 3.4 does not fail that read: it exits 0 and prints an empty `#{window_id}` with the global `remain-on-exit` value, `on`.
`OpenWindow` treated only a failed read as "gone", so it read `on` as a mismatch and returned `ErrWindowReadBack`.

Closes #761

## Design decisions

- The read-back now asks for `#{window_id} #{remain-on-exit}` (`windowReadBackFormat`).
  A failed read, or a reply whose window id is not the new window's, counts as the window being gone, and `OpenWindow` returns the opened window.
- Only a reply carrying the new window's own id and a value other than `off` is still a mismatch.
  That path kills the window by id and wraps `ErrWindowReadBack`, so the bound of the widened outcome is that a live new window always answers with its own id.
- A tmux version that fails the read on a missing target keeps working through the failed-read branch.
- The mismatch error now quotes the parsed value instead of the whole reply.
- `internal/reedengine/doc.go` records the read-back rule.
- No change to the tmux-tier test, which already covers the exit-at-once case.
- Unit tests: the fake-driven `TestOpenWindow` rows answer in the new format, and a new row covers a reply from the global scope (empty id, `on`).
  `TestEnsureShellWindow` fakes the new format too.

## To check by hand

- The tmux-tier `TestOpenWindow_RealTmux` was not run here.
  Run it on a machine with tmux, ideally 3.4 and 3.5, to confirm how each resolves a vanished `-t` target.
  The tmux 3.4 behavior above was checked by hand on a private socket.

# OpenWindow counts a read-back answered by no window as gone

Fixes #761.

`OpenWindow` reads `remain-on-exit` back from the window it just opened.
When the window's command exits at once, the window can close before that read-back.
On tmux 3.4, `display-message -p -t @<n>` for a window that no longer exists does not fail: it exits 0, prints an empty `#{window_id}` and answers with the global option value.
With the session's `remain-on-exit` on, `OpenWindow` read `on`, tried to kill the vanished window and returned `ErrWindowReadBack`, which is how `TestOpenWindow_RealTmux/ClosesWhenItsCommandExitsAtOnce` flaked.

## Change

- `internal/reedengine/window.go`: the read-back asks for `#{window_id} #{remain-on-exit}` through the new `windowReadBackFormat`.
  A failed read or a reply whose window id is not the new window's counts as the window being gone, and the op returns the opened window.
  Only a reply from the new window itself with a value other than `off` still kills it and wraps `ErrWindowReadBack`.
- `internal/reedengine/doc.go`: the detached-windows section states the id check and why.
- `internal/reedengine/window_test.go`: the fake answers carry the window id, and a new row pins a global-scope reply (empty id, `on`) as an opened window with nothing killed.
  `windoworder_test.go` answers the new format.

## Design decisions

- The id check, not a retry or delay, decides "gone": a live window always answers with its own id, so a real mismatch is still caught, and the test stays free of timing.
- The tmux behavior was confirmed by hand on tmux 3.4 against a private socket; the tmux-tier test is unchanged, since it already covers the exit-at-once case.

## Verification

- `lyx gate test ./internal/reedengine` is green.
- The tmux tier was not run in this session; the Publish run's tmux tier exercises `TestOpenWindow_RealTmux`.

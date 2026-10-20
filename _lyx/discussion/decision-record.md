# Decision record — openwindow-exit-race

## Goal

`OpenWindow` reports a window whose command exited at once as opened, never as a `remain-on-exit` read-back mismatch, so `TestOpenWindow_RealTmux/ClosesWhenItsCommandExitsAtOnce` is deterministic.

## Scope

- `internal/reedengine/window.go`: the read-back after `new-window` in `OpenWindow`.
- `internal/reedengine/window_test.go`: the fake-driven rows that pin the read-back.
- No change to the tmux-tier test, which already covers the exit-at-once case and only flaked on the read-back.

## Decisions

### The read-back names the window it answers for

Decision: the read-back asks `display-message -p -t <window id>` for `#{window_id} #{remain-on-exit}`, and a reply whose window id is not the new window's counts as the window being gone, the same as a failed read.
Only a reply carrying the new window's id and a value other than `off` is a read-back mismatch.
Rationale: tmux 3.4, checked by hand on a private socket, answers `display-message -t @99` for a missing window with exit 0, an empty `#{window_id}` and the global `remain-on-exit` value, so a failed read alone never detects the vanished window.

### Bound of the widened "gone" outcome

Decision: the widened outcome skips the mismatch kill and error only for a reply that does not carry the new window's id.
A live new window always answers with its own id, so its mismatch is still caught and killed.

## Constraints

- `PATTERN-test-speed` and `PATTERN-test-economy`: the new behavior is a row in `TestOpenWindow`, not a new test.
- `PATTERN-comment-line-breaks`: comments one sentence per line.

## Auto-mode assumptions

- A tmux version that does fail the read on a missing target keeps working through the existing failed-read branch; both outcomes stay handled.

## Open risks

- The tmux-tier test cannot run in a darn session, so the fix is verified against tmux 3.4 by hand and by the fake-driven unit rows; the Publish run's tmux tier is its first real-tmux run.

## Acceptance criteria

- A read-back reply with an empty or foreign window id and `on` returns the opened window with no error and kills nothing.
- A read-back reply with the new window's id and `on` still kills the window by id and wraps `ErrWindowReadBack`.
- A failed read-back still returns the opened window.
- `lyx gate test ./internal/reedengine` is green.

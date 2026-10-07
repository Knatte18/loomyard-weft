# Card 1 — reed wait-mark op

**Edit:**
- `internal/reedengine/io.go`
- `internal/reedengine/io_test.go`
- `internal/reedengine/doc.go`
- `PATTERN.md`

**Intent:**
Add the one reed engine op that sets or clears a wait mark on a strand's pane (D2), as a method on `Engine` in `io.go` beside `SendText`: `func (e *Engine) SetWaitMark(guid, label string, start time.Time) error`.
It takes the op lock, requires the session, resolves the strand's pane through `resolvePaneInThisSessionLocked` exactly as `SendText` does, and then:

- a non-empty label sets two pane user options with `set-option -p -t <pane>`: `@lyx_wait` to the label and `@lyx_wait_start` to `start` as epoch seconds;
- an empty label unsets both with `set-option -p -u -t <pane>`.

An unknown guid, a strand with no pane, or a tmux failure is a returned error; the op never logs-and-swallows, so its callers (shuttle in card 11, the landing callback in card 13) decide that a failure is logged and changes no outcome.
The mark is display only: reed stores nothing in its state file for it, and the options die with the pane, so a reborn strand never carries a stale mark.

`io_test.go` pins, over the fake tmux, the exact argument lists for a set and a clear, and the error for an unknown guid.
`doc.go` gains a short paragraph on the wait mark: the two option names, that it is display only, and that the status line (card 2) renders it.

`PATTERN.md` gains one entry under "Agents and prompts", per D8:
`PATTERN-wait-mark-display-only` — Marking a wait on screen: the `@lyx_wait` and `@lyx_wait_start` pane options are set only through reed's `SetWaitMark`, by `shuttleengine` and the landing verify callback, and no Go decision reads them.
The line stays within the overview's line-length cap.

**ImpactSummary:** One new reed method and one PATTERN entry; no existing caller changes.

# Support log — openwindow-exit-race

## Interview

A darn run holds no interview; the writer read the board entry and `internal/reedengine/window.go`, and probed tmux 3.4 by hand.

## Rejected alternatives

- Dropping the read-back: rejected, since it is the only check that the chained `set-option` landed on the new window.
- Reading back only when the pane is live, via `#{pane_dead}`: rejected, since a vanished target still resolves to another window and its `pane_dead` says nothing about the new one.
- Waiting or retrying before the read-back: rejected, since it adds wall-clock dependence and still races.

## Review rounds

_No rounds yet._

## Question ledger

- Does tmux 3.4 fail `display-message -t <vanished window id>`? Resolved: no; it exits 0 with an empty `#{window_id}` and the global option value, checked on a private socket.
- Should the tmux-tier test change? Self-picked: no; it already covers the exit-at-once case.

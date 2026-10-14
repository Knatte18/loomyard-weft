# Run window does not follow the newest agent window, sticks on Driver

`reed-window-follow` · Note · bug, reed

## Seen (operator, 2026-10-14)

The operator's run window does not follow the active window, the one whose agent spawned last.
It typically stays on the Driver window while later agents spawn and work in windows of their own.

`internal/reedengine/attach.go` picks the window once, at attach: a trailing `select-window` to the newest live agent window, else the overview.
Nothing moves an attached client when a later strand spawns.

## Direction

The operator remarked that the LLM Driver is to be dropped anyway, replaced by Go plus a spawnable watcher (see `orch-split` and the Go driver with the tackler behind it).
With no Driver window, the window would no longer stick there, but a client attached before a spawn still needs to follow the newest agent window.

## Open

- Whether reed moves attached clients to a new strand's window on spawn, and when that should not happen (the operator has picked a window by hand).
- Whether this waits for the Driver to go, or is fixed in reed now.

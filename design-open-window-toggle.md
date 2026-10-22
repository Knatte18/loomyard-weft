# A status-bar button turns automatic terminal windows for runs on or off

`open-window-toggle` · Note · enhancement, batten, reed

## Seen

Whether batten opens a desktop terminal window on a task worktree is decided by `batten.yaml`'s `open_window`, read from the template and the board default layer only, and `lyx batten run --window` forces it on whatever that says.
The operator has no quick way to turn the windows off, although reed already holds every run in one instance: each run is a window of the prime's reed session.

## Idea

A toggle the operator flips with one click, a button in the orch tab's status bar beside `follow`, that says whether runs open their terminal window automatically.
When it is off, `Open-Window` opens nothing, `--window` included, and the operator follows runs inside reed.
The `follow` button keeps its state in the tmux user option `@lyx_follow` (`internal/reedengine/bindings.go`), which dies with the tmux server; this switch must survive a restart, so it needs a machine-local home batten can read.

Operator wish, 2026-10-22; the design is open.
Since `machine-handover` landed, a resumed run reopens its window when no client is attached to the pair's reed session; this switch would govern that reopen too.

# Run windows show the board handle in their titles

`window-titles-show-handle` · Task · enhancement, reed, batten, termwindow · draft

The operator names runs by board handle (`T4`, `P1`), but no window shows it: a run's Konsole window is titled with the worktree directory's base name, the slug, and its tmux overview window is named `overview` (operator, 2026-11-03).

## Expected

- The desktop terminal window batten's Open-Window row opens for a task run is titled `<handle>: <slug>`, for example `T4: webster-subpath-anchor`.
  The title reaches `internal/termwindow` through `lym reed window --title`, or the call Open-Window makes, as today.
- A loom run's `loom-status` overview window is named `<handle>: overview`, for example `T4: overview`, in place of the bare `overview` default (`internal/reedengine/windoworder.go`).
- A beam's runs strand overview window, named `beam: <slug>` today, gains the beam project's handle the same way: `P1: beam: <slug>`.
- The handle is read from the board when the window opens.
  When the board cannot be read, or the entry carries no handle, each title falls back to today's name and the run goes on; a missing handle never fails a row.
- A window opened by hand with `lym reed window` and no `--title` keeps the directory's base name.
- Tests pin the titled and the fallback names of both windows.

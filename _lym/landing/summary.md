# Run a configured window command after lym opens a window

A window lym opens, a run's terminal window (`lym reed window`) or a VS Code window (`lym ide spawn`, `lym ide open`), lands wherever the window manager puts it.
On a tiling desktop the operator then moves each by hand.
Lym now runs a machine-local command after it opens a window, so the operator's own tiling script can snap the new window into the active desktop's grid.
Lym stays ignorant of KDE or any other window manager: with the command unset, the default, nothing runs.

## What changes

A new module, `internal/desktophook`, owns a hub-wide `desktop.yaml` with two keys, both machine-category, so each machine sets its own with `lym config desktop --layer machine --set <key>=<value>` and the local layer refuses them:

- `window_command`: the shell command to run; empty or blank runs nothing.
- `window_wait_ms`: milliseconds to wait after the window opens before the command runs, so the script sees the new window (template default 1500).
  A negative value is refused, naming the key.

`desktophook.AfterWindow` waits, then runs the command through the platform shell in the window's directory.
The run is bounded by a timeout and an output-read wait delay, so a hung script cannot hold up the verb that opened the window.
A failure to load the config, start the command, a nonzero exit or a timeout returns an error carrying the command's output.

The three callers only warn on that error, since the window is already open:

- `lym reed window` runs the command after the launcher starts, under the config of the worktree holding the told directory.
  A directory in no worktree has no hub config to name a command, so nothing runs.
- `lym ide spawn` and `lym ide open` (`OpenTask`) run it after VS Code launches, and not after a failed launch.
  For the prime, the window's directory is the hub directory, because the launch path is the hub workspace file.

`configreg`, the config docs and `docs/overview.md` register the new module.

## Design decisions

- The wait is a fixed delay, not a poll for the window to map, because lym knows nothing about the desktop to ask.
  The key lets a machine lengthen it.
- The command runs in the window's directory so a script can tell which worktree's window it was started for.
- The timing (sleep, command timeout, wait delay) is held in a value inside `desktophook` so tests shorten it; `reedcli` has a `placeWindow` seam for the same reason.

## Check by hand

Not run: the command against a real KDE session.
With `lym config desktop --layer machine --set window_command=~/.local/bin/arrange-to-tiles.sh`, run `lym reed window <dir>` and `lym ide open <slug>` and confirm the new window is tiled.
Also check that a wait of 1500 ms suffices for Konsole and VS Code to map on that machine.

Findings left open for the reviewer to check:

- `OpenTask` and a failed launch in `internal/ideengine/spawn.go`: confirm `spawn_hub_integration_test.go` pins that `OpenTask` runs the command and that a failed launch runs none.
- The prime's window directory in `internal/ideengine/spawn.go`: the hub directory is derived from the workspace file's directory; confirm it is right where the hub directory is a file or the workspace file is absent.
- The command timeout in `internal/desktophook/window.go`: confirm the 30-second `commandTimeout` and 5-second `commandWaitDelay` are exercised by a test through the `timing` seam rather than only by reading.
- The wait in `internal/desktophook/window.go`: confirm a test pins that `window_wait_ms` reaches `sleep` and that an empty command skips the wait.

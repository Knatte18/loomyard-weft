# Run the machine's window command after lym opens a window

After lym opens a desktop window, it now runs a command named per machine, so the operator's own tiling script can snap the new window into the active desktop's grid.
This covers the terminal window of `lym reed window`, including batten's Open-Window row, which calls it, and the VS Code window of `lym ide spawn` and `lym ide open`, including batten's Open-IDE row and `lym ide menu`.
Lym knows nothing about KDE or any other window manager.
With the command unset, the default, nothing runs.

## What changes

- A new hub-wide config module, `desktop` (`internal/desktophook`), has two keys:
  - `window_command`: the shell command to run. The default is empty.
  - `window_wait_ms`: how long lym waits before it runs the command. The default is 1500.
  Both keys are classified `CategoryMachine`, so they go in the hub's machine override, which git does not track: `lym config desktop --layer machine --set window_command=~/.local/bin/arrange-to-tiles.sh`, run from the prime.
  The local layer refuses both keys, as it refuses every machine key.
- `desktophook.AfterWindow` loads the config, waits `window_wait_ms` and then runs `window_command` through the platform shell (`sh -c`, or `cmd /C` on Windows) in the window's directory.
  A run is bounded at 30 seconds, and its spawn and end are logged.
  It returns an error that carries the command's output.
- `lym reed window` calls it after the launcher starts, under the config of the worktree that holds the directory it was given.
  If that directory is in no worktree, nothing runs.
  `ideengine.Spawn` and `OpenTask` call it after `code` starts.
  `Seed` opens no window, so it runs nothing.
  The window is already open when the command fails, so a failure is logged as a warning and the verb still succeeds.

## Design decisions

- **A wait, not a check that the window is up.** The record allows either.
  Checking that a window is mapped would need X11, Wayland or KWin calls, and lym knows nothing about the window manager.
  How long a window takes to appear depends on the machine (`code` hands over to a running instance), so `window_wait_ms` is a machine key beside the command.
- **A new hub-wide module.** Only hub-wide modules have a machine layer.
  None of the existing hub-wide modules (fabric, board, gate and the others) is about the desktop, and reed's config is per-worktree.
- **The machine category now covers a desktop fact.** Until now, `CategoryMachine` meant only a bound on a resource that every worktree on the machine shares.
  It now also covers a fact of the machine's desktop.
  The definition is updated in `pattern/PATTERN-config-layers.md`, `configengine`'s doc and `CategoryMachine`'s comment.
  The command is executable, but the machine file lives in the Bolt checkout's `.lym`, which only the hub can reach and no task worktree can write, so keeping it out of the local layer is enough.
- **Missing config falls back to the template.** `desktophook` calls `LoadOrTemplate`, so a hub that has not been reconciled yet still opens windows, and so does a directory outside any hub, where `lym reed window` still works.
  It is added to the pinned set of callers that fall back, in `cmd/lym/configstrictness_test.go` and the PATTERN background.
- **The command runs to completion before the verb returns,** so a failure can be logged.
  The verb therefore returns after the wait plus the script's run time, rather than a detached shell that would report nothing.
- To keep `lym config`'s help text under its 2000-byte cap now that it lists one more module, one sentence in it is shorter.

## Tests

- `internal/desktophook`, tier0: `LoadConfig` returns the template defaults, reads both keys from a machine override, refuses a negative wait and refuses the keys in a local override.
- `internal/desktophook`, tier1: `AfterWindow` runs nothing when the command is unset, runs a set command in the window's directory and reports the output of a command that fails.
- `internal/reedcli`, tier0: the window verb runs the command once after a window opens and never when the open fails, and a failing command still gives an ok result.
- `internal/ideengine`, tier1: the hub scenario's task step checks that `Spawn` runs the command and `Seed` does not.
- `internal/configreg`: `desktop` is registered as a hub-wide module whose machine keys are `window_command` and `window_wait_ms`.

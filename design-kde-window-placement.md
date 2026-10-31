# A window lym opens is snapped into the active desktop tiling by a configured command

`kde-window-placement` · Note · feature, reed, ide, config

## Seen (operator, 2026-10-28)

The operator runs KDE Plasma with virtual desktops and tiling.
The Konsole windows lym opens on a run's worktree (`lym reed window`, batten's terminal for each run) and the VS Code windows it opens (`lym ide`) land wherever KWin puts them, untiled and on the current desktop, so the operator moves and tiles each by hand.

## Design (operator, 2026-10-31)

Lym places no window itself: the operator's own tiling script does it.
Neve's `~/.local/bin/arrange-to-tiles.sh` (Meta+Alt+A, run through xremap) fills the active desktop's existing Custom Tiling grid with the windows on it, without changing the grid's shape.
- A machine-local config key names a command lym runs after it opens a window, such as `~/.local/bin/arrange-to-tiles.sh`: after a run's terminal window, `lym reed window` and `lym ide`.
- The window lands on the active desktop, as KWin puts it today, and the command snaps it into the grid.
- Lym knows nothing about KDE: with the key unset nothing runs, and each machine names its own script (OSL-1046 runs Plasma 5).
- Lym runs the command only once the new window is mapped, or after a short wait, so the script sees it.

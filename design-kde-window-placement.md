# Windows lym opens land tiled on the right KDE virtual desktop

`kde-window-placement` · Note · feature, reed, ide, config

## Seen (operator, 2026-10-28)

The operator runs KDE Plasma with virtual desktops and tiling.
The Konsole windows lym opens on a run's worktree (`lym reed window`, batten's terminal for each run) and the VS Code windows it opens (`lym ide`) land wherever KWin puts them, untiled and on the current desktop, so the operator moves and tiles each by hand.

## Idea

A window lym opens lands tiled on the right virtual desktop by itself.
How the placement rules are written and kept is open (operator, 2026-10-28); the candidates:

- KWin window rules (`kwinrulesrc`) that match the window title or class lym already sets, such as the reed window's `--title` and the VS Code window title `lym ide seed` writes, and assign desktop and position.
  KWin keeps them, and lym only names its windows predictably; lym could write or check the rules from a config of its own.
- lym places each window after opening it, through KWin's D-Bus or a KWin script, from a config that maps a window kind (orch, run, IDE) to a desktop and a tile.
- Plasma 6 tiling: whether a rule or a script can put a window into a named custom tile, or only on a desktop and screen edge, is to be found out first.

Whatever the route, lym hard-codes no desktop environment: the placement is a config choice that does nothing on a machine without KDE.

## Open

- Which windows: run terminals only, or the orch's and VS Code too.
- The mapping: one desktop per run, per hub, or per window kind.
- Whether lym writes the rules or the operator does, with lym only keeping its window titles stable.

# Windows lym opens remember their virtual desktop and tile, and new windows land where their kind belongs

`kde-window-placement` · Note · feature, termwindow, desktop

## Wish (operator, 2026-11-26)

The operator runs KDE Plasma with ten virtual desktops and tiles the windows lym opens across them.

- lym remembers, for each window it opens, the virtual desktop it sits on and its place in the tiling.
  When the window is opened again, after a restart or a `lym machine start`, it lands straight back in that place.
- A new window knows where it belongs, by a rule the operator chose.
- The "start every hub" desktop launcher is of no use until this works: it opens every hub's windows with no control over where they land.

## Decided (operator, 2026-11-27)

- Plasma 6 only for now; OSL-1046 moves to Ubuntu 26.04 and Plasma 6 later.
  Other desktops and Plasma 5 get no placement, and opening a window there works as today.
- Tiling is the operator's own KDE Custom Tiling grids, not quick tiling:
  xremap binds Meta+Alt+N to `~/.local/bin/set-tiling-grid.sh 1 N` and Meta+Alt+Shift+N to `set-tiling-grid.sh 2 N`,
  which rebuild the current desktop's custom tiles (Meta+T) into a rows by columns grid through KWin scripting (`workspace.rootTile`, `Tile.split`),
  then `arrange-to-tiles.sh` fills the zones in reading order with the desktop's windows.
  A window's place is therefore a virtual desktop and a zone of that desktop's grid; a new window takes a free zone.
- Two behaviours for where a starting run's window lands:
  1. On the orch's own virtual desktop: for small projects with one or two runs at a time under one orch.
  2. On another virtual desktop than the orch's, in practice the orch's desktop plus one:
     the loomyard hub's orch sits on B1 and every run lands on B2.

## To settle

- How the behaviour is chosen: a hub default, a setting on a beam or project on the board, or both.
- Whether the rule keys on the parent orch's desktop (relative, "same" or "next") or names a desktop.
- What happens when the target desktop's grid has no free zone.
- Where the remembered places live: machine state, or hub config per machine.

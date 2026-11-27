# Windows lym opens remember their virtual desktop and tile, and new windows land where their kind belongs

`kde-window-placement` · Task · feature, termwindow, desktop · starting · Open-IDE

## Wish (operator, 2026-11-26)

The operator runs KDE Plasma with ten virtual desktops and tiles the windows lym opens across them.

- lym remembers, for each window it opens, the virtual desktop it sits on and its place in the tiling.
  When the window is opened again, after a restart or a `lym machine start`, it lands straight back in that place.
- A new window knows where it belongs, by rules the operator sets in config.
- The "start every hub" desktop launcher is of no use until this works: it opens every hub's windows with no control over where they land.

## Decided (operator, 2026-11-27)

- Plasma 6 only for now; OSL-1046 moves to Ubuntu 26.04 and Plasma 6 later.
  Other desktops and Plasma 5 get no placement, and opening a window there works as today.
- Tiling is KDE Custom Tiling (Meta+T) grids, as the operator's own scripts build them:
  xremap binds Meta+Alt+N to `~/.local/bin/set-tiling-grid.sh 1 N` and Meta+Alt+Shift+N to `set-tiling-grid.sh 2 N`,
  which rebuild the current desktop's custom tiles into a rows by columns grid through KWin scripting (`workspace.rootTile`, `Tile.split`),
  then `arrange-to-tiles.sh` fills the zones in reading order with the desktop's windows.
  A window's place is a virtual desktop and a zone of that desktop's grid.
- The operator already has an automatic script that snaps a new window into the desktop's existing tiling.
  lym goes further: it also chooses the grid for the desktop from the rules, sets it, and snaps the window into it.
- Placement is relative to the parent orch's virtual desktop, never a named desktop:
  - `same`: the run's window lands on the orch's own desktop, for small projects with one or two runs under one orch.
  - `next`: it lands on the orch's desktop plus one; the loomyard hub's orch sits on B1 and its runs on B2.
    When that desktop is full, the window goes on to the desktop after it, and so on.
  The hub sets the default per machine, and a beam or project on the board may override it.
- lym's tiling always keeps to one screen, even with two external screens attached at work or at home.
- The tiling rules live in config, and the grid depends on the screen lym tiles on, its resolution and scale:
  the laptop screen takes fewer windows per desktop than an external one, so a hub's runs spread over several desktops by the `next` overflow.
  A desktop never holds more than ten windows, so the largest grid is 2x5.

## Proposed, to confirm in Discussion

- The screen lym tiles on is the one KDE ranks priority 1 (Display Configuration, `kscreen-doctor -o` shows `priority`).
  KDE stores the priorities per set of connected screens, so the operator chooses the main screen once per setup in KDE, and lym follows it.
  On Neve today the external DP-2 is priority 1 and the laptop's eDP-1 priority 2.
- A rule states a smallest tile size in logical pixels rather than a grid per setup:
  lym derives the most rows and columns from the screen's logical size (DP-2 is 3840x2160 at scale 1.5, so 2560x1440; eDP-1 1920x1200 at 1.25, so 1536x960),
  capped at 2x5, and a desktop is full when its grid is.

## To settle

- How a desktop's grid grows with its windows, for example one column per window up to a row's width, then a second row.
- What lym does with the windows already placed when the screen setup changes.
- Where the remembered places live: machine state, or hub config per machine.
- Where the operator's existing auto-snap script lives: it is not on Neve under `~/.local/bin`, KWin scripts or `kwinrulesrc`.

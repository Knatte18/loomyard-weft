# Windows lym opens remember their virtual desktop and tile, and new windows land where their kind belongs

`kde-window-placement` · Note · feature, termwindow, desktop

## Wish (operator, 2026-11-26)

The operator runs KDE Plasma with ten virtual desktops and tiles the windows lym opens across them.

- lym remembers, for each window it opens, the virtual desktop it sits on and its place in the tiling.
  When the window is opened again, after a restart or a `lym machine start`, it lands straight back in that place.
- A new window knows where it belongs: windows of one kind share a place the operator chose.
  For example, every run window of the loomyard hub lands on the virtual desktop named "Board B2".
- The "start every hub" desktop launcher is of no use until this works: it opens every hub's windows with no control over where they land.

## To settle

- Through what lym reads and sets a window's virtual desktop and tile on KDE Plasma (KWin scripting, D-Bus), and what it does on other desktops.
- What "kind" a placement rule keys on: hub, role (orch, run strand, board session), or run.
- Where the remembered places and the rules live: machine state, or hub config per machine.

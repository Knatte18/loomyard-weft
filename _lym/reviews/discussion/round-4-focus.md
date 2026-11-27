---
round: 4
focus:
  - "D2's window match and its acceptance criterion in decision-record.md: does the resource class they require match what KWin reports for a Konsole window on Wayland (upstream source gives the app_id `org.kde.konsole`, from KWin Plasma/6.6 src/xdgshellwindow.cpp `handleAppIdChanged`), is the evidence recorded, is an unobserved value listed in Open risks, and is an X11 session either covered or excluded?"
  - "D3 and D10 script runs: KWin Plasma/6.6 src/scripting/scripting.cpp `loadScript` assigns id `scripts.size()` and returns -1 for an already loaded plugin name. Can two overlapping runs, or a script left by a killed child, still make `run` reach a script other than the caller's own, and do the stated lock nesting, timeouts, bounds and acceptance criteria agree with the chosen rule?"
  - "D3's query report against D6 and D8: does the report carry every input the pure planner and the apply script use (zone boxes and padding per desktop, a per-window identity, the current desktop, and zone membership on a non-current desktop, where `window.tile` is not set by `Tile::manage`), with each KWin fact either recorded with its evidence or listed in Open risks?"
  - "D9's renumber and its trigger: can an apply by one child change the home or order key of a reopened window whose entry is still unsettled, and do windows reopening together end in their remembered order, with a settled definition that matches how D8 and D10 use it?"
---

Each entry covers a decision whose text a fix for this round is expected to change.

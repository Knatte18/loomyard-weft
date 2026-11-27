---
verdict: BLOCKING
findings:
  - id: F5
    severity: BLOCKING
    class: design
    location: _lym/discussion/decision-record.md:275
    summary: "Open risks' list of unobserved KWin facts leaves out D2's resource class `konsole` (line 46), which was never observed (D2 cites only a pid and a caption) and is wrong on Wayland by upstream source: KWin sets resourceClass to the app_id, which for Konsole is `org.kde.konsole`, so D2's match finds no window and nothing is placed"
    fix: "In D2 (line 46) require the resource class `org.kde.konsole`, the Wayland app_id, and say so wherever D2's rationale names the three-part match. Record the evidence in Constraints: KWin Plasma/6.6 src/xdgshellwindow.cpp `handleAppIdChanged` calls `setResourceClass(resourceName(), applicationId)`; KCoreAddons src/lib/kaboutdata.cpp builds Konsole's desktop file name `org.kde.konsole` and `setApplicationData` puts it on the application; `StartupWMClass=konsole` in the desktop file is not what KWin reports on Wayland. State that it is read from source and not observed on Neve, and add it to the Open risks probe list as a read-only probe (resourceClass of a running Konsole window). If an X11 session is meant to be placed too, name the class accepted there. In the D2 acceptance criterion (line 293) name `org.kde.konsole` as the matching class."
  - id: F6
    severity: BLOCKING
    class: design
    location: _lym/discussion/decision-record.md:281
    summary: "The listed script fact assumes `run` on the returned id runs the caller's own script, but KWin 6.6 gives a script the id `scripts.size()`, so after an earlier script unloads, the next load reuses a still-loaded script's id and its `run` reaches that other script; D10 runs queries concurrently outside the lock, D3 states no unique plugin name, and D3's bound that a killed child's script is inert (line 78) is false"
    fix: "Record in Constraints what KWin Plasma/6.6 src/scripting/scripting.cpp does: `loadScript` returns -1 when the plugin name is already loaded; otherwise the id is `scripts.size()`; a script leaves the list only when its deferred deletion runs; the constructor registers `/Scripting/Script<id>` and ignores the failure when the path is taken (Qt's QDBusConnection::registerObject returns false then). Then decide in D3 and D10 how a run reaches its own script: for example, one machine-wide script lock held from loadScript to unload for every run, queries and the window wait included (replacing the claim at line 207 that a query runs outside the lock, and saying how it nests with `kwinplace.lock` and its 120-second timer), a plugin name unique per run, -1 treated as a failed load, and leftover lym scripts unloaded before a load. State what still collides with scripts lym does not own, and rewrite D3's bound on a killed child: its never-run script can be run by a later caller that gets the same id, which for an apply script moves windows by a stale plan. Add an acceptance criterion for two runs that overlap."
  - id: F7
    severity: BLOCKING
    class: design
    location: _lym/discussion/decision-record.md:278
    summary: "D6's zone reading cannot be done from the query D3 describes (line 67): it reports no zone boxes or padding, no per-window identity the apply can address (two windows can share a pid), and no current desktop for D8's anchor; and its `current tile position` has no per-desktop source in `window.tile`, which KWin 6.6 reads from the single requested tile that `manage` leaves unset on a non-current desktop"
    fix: "In D3 step 1 (line 67) have the query report: each window's `internalId`, by which the apply script addresses it; `workspace.currentDesktop`; and, for every desktop on the tiling output, the leaf tiles of `rootTile(output, desktop)` with each one's `absoluteGeometry`, `padding` and `windows`. In D6 (line 105) and D3 step 3 say that a window's zone on a desktop is the leaf whose `windows` lists it, never `window.tile`. Record in Constraints, from KWin Plasma/6.6: src/window.h declares `tile` as READ `requestedTile`; src/tiles/tile.cpp `manage` refuses a window not on the tile's desktop, adds it to the tile's windows, and calls `requestTile` or resizes only when the tile's desktop is current. Add reading `Tile.windows` on a non-current desktop to the Open risks line 278. Extend the D3 reader criterion to cover these fields."
  - id: F8
    severity: BLOCKING
    class: design
    location: _lym/discussion/decision-record.md:185
    summary: "D9's renumber takes in a reopened window whose entry is still unsettled: it sits unplaced on the desktop KWin opened it on, and when that is its home, another child's apply orders it as untiled (last, by stacking) and rewrites the remembered order keys before its own placement runs, which loses the zone order the Goal promises on a resume that opens many windows at once"
    fix: "In D9 restrict the renumber and its trigger bullet (line 182) to settled entries: a window with an unsettled entry counts as absent, as a window not yet reopened does, and keeps its home and order key. The window an apply places or settles in that same write counts as settled. Say how line 187's numbering treats it (it keeps its position among the home's entries). Extend the D9 criterion at line 308 with: another child's apply on a different desktop leaves the order key of an unsettled window sitting on its home unchanged, and windows reopening on their home desktop end in key order."
prior:
  - id: F1
    status: resolved
    answer: "D8 reads the anchor only from a settled parent entry, releases the lock and waits again within one bound, and leaves a fallback window homeless; the acceptance criteria follow."
  - id: F2
    status: resolved
    answer: "The renumber and its trigger cover only windows that sit on the desktop and are homed on it, and the D9 criterion covers the overflowed window."
  - id: F3
    status: resolved
    answer: "The parent declined the probes, and each unobserved fact now sits in Open risks with the decision resting on it; the rubric exempts a fact the writer was told not to gather."
  - id: F4
    status: resolved
    answer: "D12 runs no window_command on SIGTERM and ends one already running, and the D12 criterion covers it."
---

# Round 3 review — kde-window-placement discussion

F5 to F7 are located on the changed Open risks lines that inventory what is unverified about KWin.
Their fixes reach decisions the last round left unchanged, as each `fix` names.

The KWin evidence is upstream source on the `Plasma/6.6` branch, read through a fetch subagent.
KCoreAddons was read at `master`, not at Neve's installed version.
Nothing was run against the live KWin session.

### [BLOCKING] D2's resource class matches no Konsole window on Wayland

**Location:** `_lym/discussion/decision-record.md:275` (the wrong value is at line 46)

**Issue:** D2 matches on resource class `konsole`.
Its observation used `kdotool getwindowpid` and `getwindowname`, which read no class.
`XdgToplevelWindow::handleAppIdChanged` sets the resource class to the app_id,
and Konsole's app_id is `org.kde.konsole`.
Neve runs Wayland, so every window wait would time out.
The Open risks list does not name this fact, so the implementing run's probes would not check it.

### [BLOCKING] A script run can reach another script

**Location:** `_lym/discussion/decision-record.md:281`

**Issue:** `Scripting::loadScript` takes `const int id = scripts.size()`.
Child A loads (id n), child B loads (id n+1), A unloads, child C loads and gets id n+1 again.
C's D-Bus registration fails silently, and C's `run` on `/Scripting/Script<n+1>` reaches B.
If B is already running, C's body never runs and C times out as a failed run, so its window is not placed.
If B has not run yet, C starts B's body.
This hits the apply under the lock too, since queries from other children overlap it.
The probe "a script unloaded right after `run`" would not find this, since it drives one script alone.

### [BLOCKING] The query does not report what D6 orders by

**Location:** `_lym/discussion/decision-record.md:278` (the report's field list is at line 67)

**Issue:** The planner is pure, so the snapshot is all it has.
D6 places an untiled window by comparing its frame with a zone box inset by padding, and the snapshot holds neither.
On a screen with a bottom panel every window of a 1-row desktop is untiled (line 111), so this is the common case.
The apply has only pid, class and caption to find the windows the planner ordered,
and two windows of one other program can share all three.
`window.tile` is one value per window, and `Tile::manage` on a non-current desktop does not set it.

### [BLOCKING] A renumber rewrites the keys of windows still waiting for placement

**Location:** `_lym/discussion/decision-record.md:185`

**Issue:** Windows A (key 0) and B (key 1) are homed on the current desktop.
Both reopen: their pre-wait writes leave them unsettled, and both sit there untiled.
A third child applies on another desktop and its report shows A and B.
Their current order is stacking order; when B is below A, it disagrees with the keys, and the renumber gives B key 0.
A's and B's own placements then restore the wrong order.
Line 180 skips unsettled entries only in the hand-move rule.

## Rejected

Not raised, since this incremental round takes a new finding only at BLOCKING on a changed hunk.
Each was seen while answering the focus entries and is recorded for whoever decides to take it.

- Line 139, MEDIUM: an entry from before a restart or a closed window still reads settled.
  A homeless child that takes the lock before its parent's pre-wait write anchors on the stale last desktop without waiting,
  so lock order changes the anchor and lines 159 and 241 hold only on a first start.
- Line 146, MEDIUM: a child of a settled but homeless parent is not "placed from the fallback anchor",
  so it takes a home next to a fallback position for good.
- Line 149, MEDIUM: past the last desktop the record names neither an apply nor a settling exit (line 189),
  so what the window records as home and last desktop, and whether it settles, is open.
- Lines 189 to 190, MEDIUM: a window that appears after the window wait is settled with the old or no last desktop.
  The next apply reads it as moved by hand and overwrites its remembered home with where KWin opened it.
- Line 191, MEDIUM: `state.UpdateJSON` returns `ErrDecode` before it mutates (`internal/state/state.go`),
  so an undecodable places file is never rewritten, no entry ever settles, and each child with a parent waits 180 seconds.
  "An absent or unreadable file reads as empty" holds for reads alone.
- Line 221, MEDIUM: the hidden verb's bound says it reads only its arguments, the places file and KWin.
  D3 to D12 also have it read config, the beam chain and parent records, the worktree list,
  `kscreen-doctor` and the journal, and run the operator's `window_command`.
- Line 179, LOW: a window moved onto several or all desktops has no stated home or last desktop.
- Line 212, NIT: D10 sends a child back to the parent wait on any unsettled parent,
  without line 145's exception for a spent bound.
- Line 261, MEDIUM: no tier of `PATTERN-test-speed` runs a real desktop,
  and the Acceptance criteria name neither the contract test's intent and tier
  nor the `PATTERN-daemon-wakeups` list entry that line 144 calls for.
- Line 276 on, LOW: KWin's source already answers part of the declined list.
  `src/tiles/tilemanager.cpp` keeps one root tile per desktop,
  and `manage` on a non-current desktop only records the window (`src/tiles/tile.cpp`).
- Focus entry on overflow keys, not a defect: a renumber leaves an overflowed window's home and key alone,
  and line 187 keeps keys on a desktop distinct.
  The past-last-desktop gap above is the one open path.
- Focus entry on the parent wait, not a defect: it releases the lock and is bounded.
  A parent under `off` or overflowed from a full home anchors the child on its live window.
- Focus entry on SIGTERM, not a defect: D12 runs no `window_command` on that exit.

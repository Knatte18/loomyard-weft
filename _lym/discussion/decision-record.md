# Decision record — kde-window-placement

## Goal

On KDE Plasma 6, every terminal window lym opens for a worktree lands on a chosen virtual desktop and in a zone of that desktop's KWin custom tiling grid.
A window opened again, after a restart or a `lym machine start`, returns to the desktop and zone order it last held.
A new window goes where the rules the operator sets in config send it: on its parent orch's desktop (`same`) or the desktops after it (`next`).
lym chooses each target desktop's grid from the tiling screen's logical size and a smallest tile size, sets it, and snaps the desktop's windows into it.
On any other desktop, Plasma 5 or Windows, opening a window works as it does today.

## Scope

In:

- The terminal windows lym opens on a worktree's reed session: `lym reed window`, which the batten recipe's Open-Window row and its reopen on resume run with the default `batten.open_window_command`, and `lym machine start`'s windows.
- Desktop detection, the tiling screen, grid sizing and building, zone filling, the `same` and `next` modes with their overflow, remembered places, and the config keys and the per-task beam setting that drive them.

Out:

- VS Code windows, from `lym ide` and from `lym machine start`'s workspace launch (see Open risks).
- Placement on Plasma 5, any other desktop and Windows.
- A project-level override for a project that is not a beam (see Open risks).

## Decisions

### D1 — Placement runs only on Plasma 6 with per-desktop tiling, and never fails a window open

Placement applies only when `KDE_SESSION_VERSION` is `6`, `XDG_CURRENT_DESKTOP` names `KDE`, the resolved launcher is Konsole, and `qdbus6`, `journalctl` and `kscreen-doctor` are on `PATH`, read from the environment of the process that opens the window.
The query script also checks that `rootTile` takes an output and a desktop; when it does not, placement stops as unsupported.
With any check failing, the window opens exactly as today: the Konsole command line is today's, and no KWin call is made.
With the checks passing and the resolved mode `off`, lym records the window's place (D9) and moves nothing.
Every placement failure, timeout or unsupported finding is a warning in the log; the verb that opened the window still succeeds.

Rationale: the operator decided Plasma 6 only, with other desktops working as today.
Per-desktop custom tiling arrived in Plasma 6.4, so a capability probe in the script decides rather than a version parse.
Recording under `off` gives a run whose task or beam sets a mode an anchor for its parent (D8).
Bound: a reed tmux server started without the KDE session variables, such as one started over SSH, skips placement for every window it opens, logged once per window.

### D2 — A placed window is identified by its Konsole process, its class and its title

When D1's checks pass, Konsole is launched with `--separate`, so every lym window has its own process, and the launcher returns the started process id.
A KWin window matches a remembered entry only when its `pid` equals the entry's, its resource class is `konsole`, and its caption holds the window's lym title.
A remembered place is keyed by the hub's Bolt dir and the worktree name.

Rationale: a process id alone can be reused by another program after a reboot, and a caption alone changes with the shell; the three together name one window.
`konsole --help-all` lists `--separate, --nofork  Run in a separate process` (Konsole on Neve, Plasma 6.6.6), and the Linux detach in `internal/proc/proc_linux.go` only starts a new session, so the started process is Konsole itself.
The contract test of D3 records the started process id against the KWin window's `pid` on Neve, which pins that they match under Wayland.

### D3 — One new package, `internal/kwinplace` (new), is the only code that reaches KWin and kscreen

`internal/kwinplace` (new) owns the KWin and kscreen boundary and the placement flow.
It reaches KWin by loading a generated script through `qdbus6`, running it and unloading it, as the operator's `set-tiling-grid.sh` does.
A KWin script cannot return a value, so each script prints marker lines with console.log; every line carries a random run id, and the script's last line is an end marker.
lym reads the KWin unit's user journal from just before the run, keeps only lines with its run id, and reads again until the end marker appears, at most four times a second and for at most 5 seconds; only then does it unload the script.
A run with no end marker in time fails as a timeout.
A placement is three steps:

1. A query script reports the desktops (id, name, order), the outputs (name, logical geometry), and every normal window with its process id, resource class, caption, desktops, output, minimized state, stacking position and current tile position.
2. A pure Go planner turns that snapshot, the tiling screen, the rules and the remembered places into a target desktop, a grid and an ordered window list.
3. An apply script builds the grid and snaps the windows; it reports the resulting desktop and zone of every window.

Every child process is logged, waited on and bounded by a timeout, and the script file is a temp file under `lymtemp`.

Rationale: one module per outside boundary keeps the stand-in rule simple, and a pure planner puts every placement rule under tier-1 tests.
The journal channel is what the operator's own scripts rely on, and it needs no session-bus listener inside lym; the run id and end marker keep back-to-back runs apart, since the journal's `--since` has second granularity.

### D4 — The tiling screen is the output KDE ranks priority 1

lym reads `kscreen-doctor -j` and tiles on the enabled output with the lowest `priority` value, matched to KWin's output by name.
Only windows on that output count toward a desktop's grid, and a placed window is moved onto it.

Rationale: confirms the entry's proposal; KDE stores priorities per set of connected screens, so the operator chooses the main screen once per setup and lym follows it.

### D5 — A desktop's capacity comes from a smallest tile size

Two machine keys of the desktop module, `desktop.tiling.min_tile_width` (new) with template 500 and `desktop.tiling.min_tile_height` (new) with template 600, give the smallest tile in logical pixels; both are classified `CategoryMachine`.
The screen's size is the tiling output's logical geometry as KWin's query reports it, so scale and rotation are already applied.
Columns are `floor(width / min_tile_width)` capped at 5, rows `floor(height / min_tile_height)` capped at 2, each at least 1.
A desktop's capacity is rows times columns.
A desktop is full when its counted windows, the window being placed left out, reach capacity.

Rationale: confirms the entry's proposal, and the cap keeps a desktop at ten windows at most.
With the template values DP-2 (2560x1440 logical) takes 2x5 and eDP-1 (1536x960 logical) takes 1x3.

### D6 — The grid grows one column per window, then to two rows, in remembered order

A target desktop with `n` windows after placement, and room for `C` columns and `R` rows, gets a 1 x `n` grid while `n` is at most `C`.
Past `C`, which only a 2-row capacity allows, it gets 2 rows of `ceil(n / 2)` columns, and an odd `n` leaves the last zone empty.
Zones are numbered in reading order, top row first, left to right.
The counted windows are the normal, non-minimized windows whose desktops are exactly that one desktop and that sit on the tiling screen, lym's and others alike.
Their order: windows with a remembered entry on that desktop by their remembered order key; then the rest by current zone, untiled windows last in stacking order; a tie in order key keeps the current order.
The placed window sorts by its own remembered order key, or goes last when it has none.
The grid is built as the operator's `set-tiling-grid.sh` builds it, through `rootTile(output, desktop)`.

Rationale: settles the entry's growth question with the shape it suggested, balanced rows instead of a full top row over a sparse bottom one.
Ordering by remembered keys, not insertion at an index, rebuilds a desktop's order whatever order its windows reopen in.
Counting every window matches `arrange-to-tiles.sh`, which fills the zones with all of a desktop's windows.

### D7 — The mode is a machine key, overridable down the beam chain

`desktop.window_placement` (new) is a machine key with the values `off`, `same` and `next`, template `off`, classified `CategoryMachine`.
`beam.window_placement` (new) is a per-task beam setting with the values `""`, `off`, `same` and `next`, template `""`, classified `CategoryNone`, read through `dobbyengine`'s one lookup over `hubgeom.TaskSettingChain`.
A window's mode is the lookup's value for its worktree when that is not `""`, otherwise `desktop.window_placement` (new) from the hub's stack on this machine.
A task sets it through its board `run_settings`, and a beam through its worktree's config, as the beam's `run_watch` is set today.
Both keys refuse any other value when loaded.

Rationale: the operator decided a per-machine hub default with a beam or project override.
The per-task lookup is the existing override path down the beams, and it falls back to the hub default and template, never the machine layer, so the empty value hands over to the machine key.
Bound: a value of `beam.window_placement` (new) in the prime's own files or the hub default layer applies on every machine and outranks the machine key, as every per-task beam setting does; the per-machine default lives only in the machine key.
A project that is not a beam has no setting of its own.

### D8 — The target desktop follows the parent orch's desktop

A window that has a remembered place whose desktop still exists goes back there.
A window without one is placed by the mode:

- Its anchor is the desktop of its parent orch's window: the worktree `hubgeom.ParentOf` names, or the prime for a beam or a task under the prime.
  The parent's window is its live match under D2.
  With no live match, lym waits for one, re-reading the places file at most once a second for at most 20 seconds, since `lym machine start` and batten's run reopens open windows at the same time.
  Past that wait the parent's remembered desktop is the anchor, and with none the current desktop is.
- `same` starts at the anchor and `next` at the anchor plus one.
- A full start desktop passes the window to the desktop after it, and so on.
- Past the last desktop, the window is left where KWin opened it, and the log warns that every desktop from the start is full; that desktop may then hold more than its capacity.

The prime's own window, having no parent, starts at the current desktop under either mode.
A remembered desktop that is full overflows the same way from that desktop.
A desktop at or over capacity is never a target, so its grid is never rebuilt.

Rationale: the operator decided placement relative to the parent orch's desktop, with `next` overflowing to the desktops after it; `same` takes the same overflow so a full desktop never gets another window.
lym creates no desktops: the operator keeps ten.

### D9 — Remembered places are a per-hub cache in `.lym`

The places live in one JSON file under the hub's `BoltDir(hub)/.lym`, its path named by `lymdirs`, written through `internal/state` and tolerant of unknown keys.
Each entry holds the worktree name, the window's process id, its lym title, the KWin desktop id, an order key within that desktop, the output name and the time recorded.
A placement first records the new window's process id and title, before any wait.
After each apply, and after a record-only run under `off`, lym updates the entries of every window of this hub the report shows:

- a window whose desktop changed, or whose order on its desktop disagrees with the entries' order keys, gets the desktop's windows renumbered in their current order;
- otherwise each entry keeps its order key, so a window not yet reopened keeps its place in the order;
- a window placed without a key gets the desktop's highest key plus one.

Entries for a worktree the hub no longer has are dropped at that write.
An absent or unreadable file reads as empty: every window is then placed as new.

Rationale: settles the entry's question in favour of machine state; places are observed facts of one machine, not settings, and a fresh machine starts without them, so `PATTERN-lym-files` puts them under `.lym`.
Recording at every placement catches a window the operator moved by hand, without a watcher in KWin.
Bound: a placement by one hub that rearranges another hub's windows on a shared desktop leaves that hub's order keys stale until its own next placement.
A second `lym machine start` that opens a second window on a worktree points its entry at the new window; the old one counts as an ordinary window until it is closed.
A `window_command` that moves windows after placement leaves the entries stale until the next placement.

### D10 — Placement runs in a detached child, and a machine-wide lock serializes the KWin steps

The opening verb starts the window and hands its placement to a detached child of the lym binary, through a hidden verb that takes the hub, the worktree name, the title and the process id, and returns at once.
The child records the window (D9), waits for it to appear in KWin, polling at most four times a second for at most 10 seconds, and waits for the parent's window (D8), all outside the lock.
It then takes one per-user lock through `internal/lock`, waiting at most 120 seconds, and runs the query, plan, apply and record steps under it.
A child that cannot take the lock in time, or whose window never appears, logs a warning and exits.
The child is started and logged as lym's other detached children are, its binary named through `selfexec.LymBinary`, and registered in `internal/runproc` when the opening verb runs inside a run.

Rationale: the batten Open-Window row bounds its command at 30 seconds, and a resume after `lym machine start` opens many windows at once, so placement inside the verb would fail rows.
The "start every hub" launcher opens many windows together, and two placements planning from one snapshot would fill the same zone.
Bound: the hidden verb moves only windows on the target desktop and the placed window, and acts only on a process id whose KWin window matches D2; it reads no input but its arguments, the places file and KWin.

### D11 — A screen change rearranges nothing on its own

When the screen setup changes, lym moves no window by itself.
The next placement on a desktop with room builds that desktop's grid for the current tiling screen, ordered by the remembered keys.
Only the target desktop's windows are rearranged by a placement.

Rationale: settles the entry's screen-change question with the least surprise: lym never touches a desktop the operator is not opening a window on.

### D12 — Every window-opening path places through one call

`lym reed window` and `lym machine start` both hand each window they start to the one placement call of D10.
When placement applies, the child runs `window_command` after its placement, with no `window_wait_ms` sleep, since it has already seen the window; when it does not apply, `lym reed window` runs `window_command` as today.
`lym machine start` opens the prime's and beams' windows parents first; run windows come from batten's reopens alongside them and rely on D8's wait for the parent.

Rationale: the batten Open-Window row and its reopen run `lym reed window`, so the two paths cover every terminal window lym opens.
Keeping `window_command` lets an operator's own script still act after lym places the window; it may rearrange windows lym placed.
Bound: a hub whose `batten.open_window_command` no longer runs `lym reed window` gets no placement for its run windows.

## Constraints

- Plasma 6.6.6, KWin 6.6.6, Wayland on Neve (`plasmashell --version`, `kwin_wayland --version`, `XDG_SESSION_TYPE`).
- KWin 6.6's scripting API (KWin's workspace_wrapper.h and tile.h on the Plasma/6.6 branch): the workspace's `rootTile(output, desktop)`, with `tilingForScreen` deprecated since 6.3; a tile's `manage`, `unmanage`, `relativeGeometry`, `tiles` and `windows`; a window's `pid`, `resourceClass`, `caption`, `desktops`, `output`, `minimized`, `normalWindow` and `tile`; the workspace's `desktops`, `currentDesktop`, `windowList()`, `stackingOrder` and `screens`.
  `split` and `remove` come from the custom tile class and are used by the operator's working `set-tiling-grid.sh`.
- The operator's scripts load a script with `qdbus6 org.kde.KWin /Scripting loadScript <file> <plugin>`, run it through its `/Scripting/Script<id>` object and unload it by plugin name.
- A KWin script returns nothing to its caller; console.log lines land in the `plasma-kwin_wayland.service` user journal, which `arrange-to-tiles.sh` reads with `journalctl --user -u` and `--since`.
- `kscreen-doctor -j` carries `priority` and `scale` per output; on Neve DP-2 is priority 1 at scale 1.5 and eDP-1 priority 2 at 1.25.
- lym depends on no operator auto-snap script; none exists on Neve.
- `PATTERN-test-stand-ins`: tests stand in for `qdbus6`, `journalctl` and `kscreen-doctor` only below `internal/kwinplace` (new), with recordings taken from Neve's live session and naming their source, a `tier0` test running the readers over every recording, and a contract test in the tier that runs a real desktop, whose run is the operator's call.
- The closed-area entry `area-host-exits` later moves Konsole launching behind a supervisor window verb inside an area; D2's process id and D10's child must then cross that verb, which is that entry's work.

## Auto-mode assumptions

Every decision is an auto-pick of this autonomous run; the support log's question ledger names each pick.

## Open risks

- The entry says a beam or a project may override the default.
  A project entry that is not a beam carries no settings: the board refuses `run_settings` on a project.
  D7 gives the override to task and beam worktrees only; a project-level override needs the board to accept a setting on a project, which is a board design call for the parent.
- The entry's title covers every window lym opens, and VS Code windows are left out: VS Code runs all its windows in one process, so D2's match cannot find one.
  The parent decides whether VS Code placement belongs here, through a caption match, or in a later entry.

## Acceptance criteria

- D1: with `KDE_SESSION_VERSION` unset, another desktop named, a missing tool or a launcher other than Konsole, the Konsole command line is today's and no KWin call is made; a placement error is a logged warning and the verb exits zero.
- D1: under mode `off` with the checks passing, the window's place is recorded and no apply runs; a query report saying `rootTile` takes no desktop stops placement as unsupported.
- D2: with the checks passing the Konsole command line carries `--separate` and the launcher returns the process id; a snapshot window with the right `pid` but another resource class or caption does not match.
- D3: over recorded `qdbus6`, `journalctl` and `kscreen-doctor` output, the readers parse the snapshot and the apply report; a `tier0` test runs them over every recording.
- D3: lines of another run id are ignored, and a journal with no end marker within the bound fails the run as a timeout.
- D3: the planner, given a snapshot, rules and places, returns the target desktop, grid and window order; tier-1 tests cover each rule in D4 to D9 through it.
- D4: the output with the lowest priority among enabled outputs is the tiling screen, and windows on other outputs are not counted.
- D5: a 2560x1440 logical output gives capacity 2x5 and a 1536x960 one gives 1x3 with the template sizes; a tiny minimum still caps at 2x5; a desktop holding capacity minus one windows plus the placed window is not full.
- D6: 3 windows with 5 columns give 1x3; 6 give 2x3; 7 give 2x4 with the last zone empty; windows with remembered keys 2, 0 and 1 reopening in that order end in key order.
- D7: a task `run_settings` value wins over its beam's, a beam's over the prime's, `""` falls to the machine key, and an unknown value is refused on load.
- D8: `next` from desktop 1 with desktop 2 full lands on 3; past the last desktop nothing is moved and the warning names the start desktop; a remembered desktop wins over the mode; a parent window recorded during the wait anchors its child; past the wait the parent's remembered desktop, then the current one, is the anchor.
- D9: after an apply, the places file holds every reported window of the hub; a reopen that keeps the keys' order keeps the keys, a hand move renumbers the desktop; a worktree the hub no longer has is dropped; an absent file reads as empty.
- D10: the opening verb returns before the window appears in KWin; two children started together run their KWin steps one after the other; one that cannot take the lock in time, or whose window never appears, logs a warning and exits.
- D11: a placement touches no desktop but its target, and a desktop at or over capacity is never rebuilt.
- D12: `lym reed window` and `lym machine start` both reach the one placement call; with placement applying, `window_command` runs after placement with no sleep; without, it runs as today; `lym machine start` opens a parent's window before its children's.
- Review: the desktop template's comments for the new keys name the machine-layer set command, and `PATTERN-config-layers`'s background lists `beam.window_placement` (new) among the per-task beam settings.

## Notes for the plan writer

- The operator's `~/.local/bin/set-tiling-grid.sh` and `arrange-to-tiles.sh` are working references: the first releases the tiled windows, removes the root tile's children, splits columns then rows and sets each tile's `relativeGeometry`; the second filters windows and calls `manage` per zone.

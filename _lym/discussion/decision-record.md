# Decision record — kde-window-placement

## Goal

On KDE Plasma 6, every terminal window lym opens for a worktree lands on a chosen virtual desktop and in a zone of that desktop's KWin custom tiling grid.
A window opened again, after a restart or a `lym machine start`, returns to the desktop and zone order it last held.
A new window goes where the rules the operator sets in config send it: on its parent orch's desktop (`same`) or the desktops after it (`next`).
lym chooses each target desktop's grid from the tiling screen's logical size and a smallest tile size, sets it, and snaps the desktop's windows into it.
On an X11 session, any other desktop, Plasma 5 or Windows, opening a window works as it does today.

## Scope

In:

- The terminal windows lym opens on a worktree's reed session: `lym reed window`, which the batten recipe's Open-Window row and its reopen on resume run with the default `batten.open_window_command`, and `lym machine start`'s windows.
- Desktop detection, the tiling screen, grid sizing and building, zone filling, the `same` and `next` modes with their overflow, remembered places, and the config keys and the per-task beam setting that drive them.

Out:

- VS Code windows, from `lym ide` and from `lym machine start`'s workspace launch (see Open risks).
- Placement on Plasma 5, an X11 session, any other desktop and Windows.
- A project-level override for a project that is not a beam (see Open risks).

## Decisions

### D1 — Placement runs only on Plasma 6 with per-desktop tiling, and never fails a window open

The pre-launch checks read the environment of the process that opens the window, and they decide the Konsole command line.
They pass when `KDE_SESSION_VERSION` is `6`, `XDG_CURRENT_DESKTOP` names `KDE`, `XDG_SESSION_TYPE` is `wayland`, the resolved launcher is Konsole, and `qdbus6`, `journalctl` and `kscreen-doctor` are on `PATH`.
With any pre-launch check failing, the window opens exactly as today: the Konsole command line is today's, and no KWin call is made.
With them passing, Konsole starts with `--separate` (D2) and the detached child of D10 takes the window over.
The child's first query script runs the post-launch per-desktop tiling probe (Open risks).
When the probe fails, the window stays where KWin opened it, its entry is settled there unplaced (D9), and the log warns that placement is unsupported.
With every check passing and the resolved mode `off`, lym records the window's place (D9) and moves nothing.
Every placement failure, timeout or unsupported finding is a warning in the log; the verb that opened the window still succeeds.

Rationale: the operator decided Plasma 6 only, with other desktops working as today.
Placement takes a Wayland session only: KWin reports Konsole's resource class differently on X11 (D2), and only Wayland runs on Neve.
Per-desktop custom tiling arrived in Plasma 6.4, so a capability probe in the script decides rather than a version parse;
the probe can run only once KWin is reached, after Konsole has started.
Recording under `off` gives a run whose task or beam sets a mode an anchor for its parent (D8).
Bound: a reed tmux server started without the KDE session variables, such as one started over SSH, skips placement for every window it opens, logged once per window.

### D2 — A placed window is identified by its Konsole process, its class and its title

When D1's pre-launch checks pass, Konsole is launched with `--separate`, so every lym window has its own process whatever the operator's Konsole settings, and the launcher returns the started process id.
A KWin window matches a remembered entry only when its `pid` equals the entry's, its resource class is `org.kde.konsole`, the app_id Konsole sets on Wayland, and its caption holds the window's lym title.
A remembered place is keyed by the hub's Bolt dir and the worktree name.

Rationale: a process id alone can be reused by another program after a reboot, and a caption alone is shared by a second window opened on the same worktree; the three together name one window.
`konsole --help-all` lists `--separate, --nofork  Run in a separate process` (Konsole on Neve, Plasma 6.6.6), and the Linux detach in `internal/proc/proc_linux.go` only starts a new session, so the started process is Konsole itself.
Observed on Neve on 2026-11-27: `pgrep -a konsole` lists each lym window as its own `konsole` process, started without `--separate`.
`kdotool getwindowpid` and `getwindowname`, run the same day, give the window titled `ORCH [ly:P0] root` the pid 3913 of its Konsole process and the caption `ORCH [ly:P0] root — Konsole`.
Neither tool reads the resource class: `org.kde.konsole` is read from KWin's and KCoreAddons' source (Constraints), not observed on Neve.
The desktop file's `StartupWMClass=konsole` is not what KWin reports on Wayland.
`--separate` is still passed: Konsole's "Run all Konsole windows in a single process" setting, absent from Neve's `konsolerc` and so off, would hand a new window to a running Konsole process, and every lym window would then carry that process's pid.

### D3 — One new package, `internal/kwinplace` (new), is the only code that reaches KWin and kscreen

`internal/kwinplace` (new) owns the KWin and kscreen boundary and the placement flow.
It reaches KWin by loading a generated script through `qdbus6`, running it and unloading it, as the operator's `set-tiling-grid.sh` does.
Every script run, query or apply, holds a per-user script lock, the file `kwinscript.lock` beside D10's `kwinplace.lock`, from before its load until its script has left KWin, so no two lym script runs overlap.
A child that holds `kwinplace.lock` takes the script lock inside it for each run, and no child takes them the other way round.
It takes the script lock through the same blocking acquire as D10's lock, for at most 30 seconds, and within D10's window wait for at most what remains of that wait; a run that cannot take it in time fails.
An acquire past its bound is abandoned: should it take the lock later, it releases it at once, so the child goes on without holding it.
A run's plugin name carries its run id, so no two runs share one, and a load that returns -1 fails the run.
Under the script lock, a run first unloads the plugin name an earlier holder left in the file `kwinscript.loaded` beside the lock, and waits it out as below;
a leftover still loaded after that wait fails the run before its load, and its name stays in the file.
Otherwise the run writes its own plugin name there and loads its script.
A KWin script cannot return a value, so each script prints marker lines with console.log; every line carries a random run id, and the script's last line is an end marker.
lym reads the KWin unit's user journal from just before the run, keeps only lines with its run id, and reads again until the end marker appears, at most four times a second and for at most 5 seconds.
A run with no end marker in time fails as a timeout.
lym unloads the script by its plugin name on every exit of the run: after the end marker, on the timeout and on any error after the load.
It then reads `isScriptLoaded` for that name four times a second for at most 2 seconds;
it empties `kwinscript.loaded` once the name reads as not loaded, leaves a name still loaded there for the next holder, and releases the script lock either way.
A child ended by SIGTERM unloads its script and releases the script lock before it exits.
Once `run` is called, the script's body runs to its end inside KWin whatever happens to lym, so a run ended at any point leaves the desktop as it was before the run or as the script left it.
A placement is three steps:

1. A query script reports `workspace.currentDesktop`, the desktops (id, name, order), the outputs (name, logical geometry), the per-desktop tiling probe's result, and every normal window with its `internalId`, process id, resource class, caption, desktops, output, minimized state, stacking position and frame geometry.
   For every desktop it reports the leaf tiles of `rootTile(output, desktop)` on the tiling output (D4), whose name lym writes into the generated script, each with its `absoluteGeometry`, `padding` and `windows`.
   A window's zone on a desktop is the leaf whose `windows` lists it, never the window's `tile`, which holds one value per window and which a `manage` on a desktop that is not current leaves unset (Constraints).
2. A pure Go planner turns that snapshot, the tiling screen, the rules and the remembered places into a target desktop, a grid and an ordered window list.
3. An apply script addresses every window by the `internalId` the query reported, since two windows of a program other than lym's Konsole can share a pid, class and caption.
   It sets the placed window's desktops to the target desktop alone and sends it to the tiling output with `workspace.sendClientToScreen`.
   It then builds the grid on the target desktop of the tiling output, snaps the counted windows into their zones and clamps the bottom row at the panel (D6).
   It reports the resulting desktop of every window and its zone, the leaf whose `windows` lists it, as in step 1.

Every child process is logged, waited on and bounded by a timeout, and the script file is a temp file under `lymtemp`.

Rationale: one module per outside boundary keeps the stand-in rule simple, and a pure planner puts every placement rule under tier-1 tests.
The journal channel is what the operator's own scripts rely on, and it needs no session-bus listener inside lym.
The run id keeps the lines of back-to-back and concurrent runs apart, and the end marker tells when a run has finished;
a `run` that reached another script logs no end marker of its run id and fails as a timeout.
KWin gives a new script the id that equals the number of scripts in its list (Constraints), so a load after an earlier script has left the list can take a still-loaded script's id, and its `run` reaches that script.
Holding the script lock until a run's script has left the list, and unloading a leftover first, keeps every lym run from reaching another lym script.
Bound: a child killed outright leaves its script loaded, with its body run or never run, and its plugin name in `kwinscript.loaded`; the next lym run unloads it before its own load.
Left loaded, a never-run apply script could be run by a later caller that gets its id, moving windows by a stale plan.
Scripts lym does not own, such as those the operator's scripts load, still collide: when one leaves KWin's list while a script loaded after it stays, the next load, lym's or another caller's, takes a still-loaded id.
A lym `run` that reaches another script then starts that script's body if it has not run and fails as a timeout, leaving its window unplaced; another caller's `run` that reaches a lym script can start a lym body.

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
The counted windows are the normal, non-minimized windows whose desktops are exactly that one desktop and that sit on the tiling screen, lym's and others alike; a window on several desktops is not counted.
Their order: windows whose home desktop (D9) is that desktop by their order key; then the rest by current zone, untiled windows last in stacking order; a tie in order key keeps the current order.
A window's current zone is the leaf tile whose `windows` lists it on that desktop in the snapshot (D3), never the window's `tile`, or, for an untiled window, the zone whose box inset by the tile's padding starts at the window's frame's top-left corner, as a clamped window's does.
A root tile with no children holds no zones, so on a desktop with no grid every window is untiled.
The placed window sorts by its own order key when the target is its home, or goes last otherwise.
The grid is built as the operator's `set-tiling-grid.sh` builds it, through `rootTile(output, desktop)`.
KWin's tiles fill the whole output, panel included.
After snapping, each zone whose bottom edge runs below the output's usable area, `workspace.clientArea` with the placement area as `arrange-to-tiles.sh` reads it, releases its window from the tile.
The apply script then sets that window's `frameGeometry` to the zone's box inset by the tile's padding and cut at the usable area's bottom.
On a screen whose panel sits at its bottom edge every bottom-row zone runs under it, so a 1-row desktop's windows are all clamped: each sits in its zone's box, and none is tiled, as with `arrange-to-tiles.sh`.

Rationale: settles the entry's growth question with the shape it suggested, balanced rows instead of a full top row over a sparse bottom one.
Ordering by remembered keys, not insertion at an index, rebuilds a desktop's order whatever order its windows reopen in.
Counting every window on the desktop follows `arrange-to-tiles.sh`, which fills the zones with lym's and other windows alike.
Unlike that script, which counts any window whose desktops include the desktop, lym leaves a window on several desktops untiled rather than tie it to one desktop's grid.
The clamp keeps the correction `arrange-to-tiles.sh` makes for the same overlap with `kdotool`; setting the frame inside the apply script needs no further tool.

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

A window whose entry (D9) has a home desktop that still exists goes back there.
A window without one is placed by the mode:

- Its anchor is the desktop of its parent orch's window: the worktree `hubgeom.ParentOf` names, or the prime for a beam or a task under the prime.
  The anchor is read under the lock, and only from a settled parent entry (D9).
  A settled parent entry gives the desktop of the parent's live D2 match in the snapshot, or the entry's last desktop when the parent has no live window.
  With no parent entry, or an unsettled one, lym first waits outside the lock until the parent's entry is settled.
  A child that finds the parent's entry missing or unsettled under the lock releases the lock and returns to that wait.
  The wait re-reads the places file from once a second, doubling to eight seconds while the file is unchanged.
  Its 180-second bound runs from the child's first entry into the wait, whether before the lock or from under it, and covers every return to it.
  That wait is a loop for `PATTERN-daemon-wakeups`'s list of unbounded waits.
  Once the bound is spent, a parent entry still missing or unsettled makes the current desktop the anchor.
  A window placed from that fallback anchor takes no home (D9), so its next placement anchors on its parent again.
- `same` starts at the anchor and `next` at the anchor plus one.
- A full start desktop passes the window to the desktop after it, and so on.
- Past the last desktop, the window is left where KWin opened it, and the log warns that every desktop from the start is full; that desktop may then hold more than its capacity.

The prime's own window, having no parent, starts at the current desktop under either mode.
A home desktop that is full overflows the same way from that desktop.
A desktop at or over capacity is never a target, so its grid is never rebuilt.

Rationale: the operator decided placement relative to the parent orch's desktop, with `next` overflowing to the desktops after it; `same` takes the same overflow so a full desktop never gets another window.
The parent's live window exists before its own placement has run, on the desktop KWin opened it on, and the lock grants no order between two placements.
An unsettled entry's home or last desktop is not where its parent lands: a parent under mode `off` stays where KWin opened it, a full home overflows it, and a homeless parent is placed by mode.
Anchoring only on a settled entry keeps a child from anchoring on a desktop its parent does not end on, and leaving a fallback window homeless keeps it from recording such a desktop for good.
`lym machine start` opens a hub's windows only after every beam start of the hub has returned (D12), so on a first start a run window's parent can open minutes after the run window; the 180-second wait covers that gap.
lym creates no desktops: the operator keeps ten.
Bound: a first start whose beam starts outlast the wait anchors that hub's run windows on the current desktop, homeless until their next placement.
A parent whose window lym never recorded, such as one opened by hand, or whose first child was killed outright, holds its child's placement and `window_command` for the whole wait; the child then anchors on the current desktop and stays homeless.

### D9 — Remembered places are a per-hub cache in `.lym`

The places live in one JSON file under the hub's `BoltDir(hub)/.lym`, its path named by `lymdirs`, written through `internal/state` and tolerant of unknown keys.
Each entry holds the worktree name, the window's process id, its lym title, and the process id whose placement child last settled the entry.
It also holds the window's home desktop with an order key within it, the last desktop the window was placed on or seen on, the output name and the time recorded.
An entry is settled when that settling process id equals its process id: the child of the current window has finished, whether it placed the window or not.
Every write of the places file is one `state.UpdateJSON` read-modify-write that changes only its own window's entry.
A write after an apply also changes the entries the rules below change, and drops the entries of worktrees the hub no longer has.
A placement first records the new window's process id and title, before any wait.
That write keeps the entry's home desktop, order key and last desktop, so the entry reads as unsettled until its child finishes.
After each apply, and after a record-only run under `off`, lym updates the entries of every window of this hub the report shows:

- the placed window's last desktop is where it landed, and the write settles its entry;
  where it landed becomes its home when it had none or landed on its home, except for a window placed from D8's fallback anchor, which stays homeless;
  an overflow from its home keeps its home and order key, so its next placement with room returns it there;
- any other window with a settled entry whose desktop differs from its entry's last desktop was moved by hand, and that desktop becomes its home and its last desktop;
  an unsettled entry is left as it is, since its window still waits for its own placement;
- a window that takes a new home desktop, placed or moved by hand, takes that desktop's highest key plus one;
- a desktop that a window was moved onto by hand, or whose windows with settled entries sitting on it and homed on it are in a current order that disagrees with their order keys, is renumbered;
  the window this write places or settles counts as settled;
- otherwise each entry keeps its order key, so a window not yet reopened keeps its place in the order.

A renumber gives the windows with settled entries that sit on that desktop and are homed on it, in their current order, the positions in key order that those windows held.
Every other entry homed on the desktop counts as absent and keeps its position among the home's entries: a window not yet reopened, one that overflowed elsewhere, or one whose entry is unsettled, which sits unplaced where KWin opened it until its own placement runs.
The desktop's entries are then numbered from 0 in that order, so no two entries share a key.
A window sitting on the desktop but homed elsewhere, such as one that overflowed onto it, keeps its home and order key.
A child that ends with no apply or record-only run, on a failed probe, a timeout, a failed run or SIGTERM, settles its own entry.
When a snapshot showed the window, that write sets the entry's last desktop to the desktop the window sat on, and keeps its home and order key.
An absent or unreadable file reads as empty: every window is then placed as new.

Rationale: settles the entry's question in favour of machine state; places are observed facts of one machine, not settings, and a fresh machine starts without them, so `PATTERN-lym-files` puts them under `.lym`.
Recording at every placement catches a window the operator moved by hand, without a watcher in KWin.
Many children write the file at once, a pre-wait write outside the lock beside a post-apply write under it.
`UpdateJSON` serializes only writers that all go through it, so every write goes through it and changes only its own fields.
Bound: a placement by one hub that rearranges another hub's windows on a shared desktop leaves that hub's order keys stale until its own next placement.
A second `lym machine start` that opens a second window on a worktree points its entry at the new window; the old one counts as an ordinary window until it is closed.
A `window_command` that moves windows after placement leaves the entries stale until the next placement.
A window that overflowed stays on the desktop it landed on until it is opened again, since lym moves no window by itself (D11).

### D10 — Placement runs in a detached child, and a machine-wide lock serializes the KWin steps

The opening verb starts the window and hands its placement to a detached child of the lym binary, through a hidden verb that takes the hub, the worktree name, the title and the process id, and returns at once.
The child records the window (D9), waits for it to appear in KWin, and waits for the parent's placement (D8), all outside the lock.
The window wait runs D3 query scripts until one shows the window's D2 match, for at most 10 seconds.
A query changes nothing, so it runs outside `kwinplace.lock`, holding only D3's script lock, and the next run starts at least one second after the previous one ends.
Every query writes its report to the file `kwinquery.json` beside D3's script lock.
A window-wait child that takes the script lock and finds there a report that ended after its previous query, or its wait, began reads that report and loads no script, so one query serves every window waiting at once.
The child then takes one per-user lock, the file `kwinplace.lock` in the per-user state directory `standalonestate.UserStateDir` names, where the machine record of hubs and the desktop launcher's lock sit.
It takes it through `lock.AcquireWriteLock`, a blocking acquire that wakes only when the lock frees.
A timer ends the wait after 120 seconds, and the child exits with the acquire still pending, which drops the attempt.
The timer counts the time spent waiting for the lock summed over every acquire of one child, so a child that returns to the parent wait waits at most 10 seconds for its window, 180 for its parent and 120 for the lock in all;
under the lock each script run adds at most 30 seconds for D3's script lock and 2 for its unload.
Under the lock it runs the query, plan, apply and record steps, each script run taking D3's script lock inside it, unless the parent's entry is missing or unsettled, when it releases the lock and returns to the parent wait (D8).
A child that cannot take the lock in time, or whose window never appears, settles its entry (D9), logs a warning, runs `window_command` (D12) and exits.
A child ended by SIGTERM mid-apply leaves the desktop as D3 states and settles its entry; one killed outright leaves its entry unsettled until the window's next placement.
The child is started and logged as lym's other detached children are, its binary named through `selfexec.LymBinary`, and registered in `internal/runproc` when the opening verb runs inside a run.

Rationale: the batten Open-Window row bounds its command at 30 seconds, and a resume after `lym machine start` opens many windows at once, so placement inside the verb would fail rows.
The "start every hub" launcher opens many windows together, and two placements planning from one snapshot would fill the same zone.
The lock serializes placements of every hub on the machine, so it sits in the per-user state directory; a lock under one hub's `.lym` would serialize only that hub.
The blocking acquire wakes nothing while the lock is held, as the Slack relay holder's lock wait does, so the 120-second wait adds no periodic wake under `PATTERN-daemon-wakeups`.
Bound: the hidden verb moves only windows on the target desktop and the placed window, and acts only on a process id whose KWin window matches D2; it reads no input but its arguments, the places file, the files beside D3's script lock and KWin.

### D11 — A screen change rearranges nothing on its own

When the screen setup changes, lym moves no window by itself.
The next placement on a desktop with room builds that desktop's grid for the current tiling screen, ordered by the remembered keys.
Only the target desktop's windows are rearranged by a placement.

Rationale: settles the entry's screen-change question with the least surprise: lym never touches a desktop the operator is not opening a window on.

### D12 — Every window-opening path places through one call

`lym reed window` and `lym machine start` both hand each window they start to the one placement call of D10 when D1's pre-launch checks pass.
Once the child is handed a window, it runs `window_command` on every exit path but SIGTERM, with no `window_wait_ms` sleep, since it has waited for the window itself.
That covers an exit after placement, after a record-only run under mode `off`, and after any warning that ends the child: a lock or window timeout, a failed probe, a failed run.
A child ended by SIGTERM, at any step, runs no `window_command`; it ends one already running, with every process the command started, before it exits.
So the window-command runner gains a way for its caller to cancel a running command and its process group.
A SIGTERM means the child's run or session is ending, and from `runproc.End` it is followed by SIGKILL 5 seconds later, well inside the command's 30-second bound, so a command started then could not count on finishing.
When the pre-launch checks fail, `lym reed window` runs `window_command` after `window_wait_ms` as today, and `lym machine start` runs none, as today.
`lym machine start` runs a hub's beam starts (`runBeamStarts`) before it opens any of the hub's windows (`openWindows`).
The dobbies those starts bring up reopen run windows through batten meanwhile, so run windows can open before their orch's window, and D8's parent wait covers the gap.

Rationale: the batten Open-Window row and its reopen run `lym reed window`, so the two paths cover every terminal window lym opens.
Keeping `window_command` lets an operator's own script still act after lym places the window; it may rearrange windows lym placed.
`lym machine start` opens windows only for the worktrees whose beam start succeeded, so it cannot open them before the starts.
Bound: a hub whose `batten.open_window_command` no longer runs `lym reed window` gets no placement for its run windows.
Under placement, `lym machine start`'s windows gain the `window_command` they never ran before.

## Constraints

- Plasma 6.6.6, KWin 6.6.6, Wayland on Neve (`plasmashell --version`, `kwin_wayland --version`, `XDG_SESSION_TYPE`).
- KWin 6.6's scripting API (KWin's workspace_wrapper.h and tile.h on the Plasma/6.6 branch): the workspace's `rootTile(output, desktop)`, with `tilingForScreen` deprecated since 6.3; a tile's `manage`, `unmanage`, `relativeGeometry`, `absoluteGeometry`, `padding`, `tiles` and `windows`; a window's `internalId`, `pid`, `resourceClass`, `caption`, `desktops`, `output`, `frameGeometry`, `minimized` and `normalWindow`; the workspace's `desktops`, `currentDesktop`, `windowList()`, `stackingOrder`, `screens`, `clientArea` and `sendClientToScreen`.
  `split` and `remove` come from the custom tile class and are used by the operator's working `set-tiling-grid.sh`.
- The operator's scripts act only on `workspace.currentDesktop` and `workspace.activeScreen`, so the KWin facts listed under Open risks rest on KWin's headers and source alone.
- Read from KWin's source on the Plasma/6.6 branch and not observed on Neve:
  - `src/xdgshellwindow.cpp` `handleAppIdChanged` calls `setResourceClass(resourceName(), applicationId)`, so on Wayland a window's resource class is its app_id;
    KCoreAddons `src/lib/kaboutdata.cpp`, read at `master`, builds Konsole's desktop file name `org.kde.konsole`, which `setApplicationData` puts on the application as its app_id.
  - `src/scripting/scripting.cpp` `loadScript` returns -1 when the plugin name is already loaded, and otherwise gives the script the id `scripts.size()`;
    a script leaves the list only when its deferred deletion runs, and its constructor registers `/Scripting/Script<id>` and ignores the failure when that path is taken, since Qt's `QDBusConnection::registerObject` returns false then.
  - `src/window.h` declares a window's `tile` as READ `requestedTile`;
    `src/tiles/tile.cpp` `manage` refuses a window not on the tile's desktop, adds it to the tile's `windows`, and calls `requestTile` or resizes the window only when the tile's desktop is current.
- Neve's `kwinrc` holds its `[Tiling]` groups per output UUID and desktop UUID, which suggests one root tile per output and desktop; no script has read that through the API.
- The operator's scripts load a script with `qdbus6 org.kde.KWin /Scripting loadScript <file> <plugin>`, run it through its `/Scripting/Script<id>` object and unload it by plugin name.
- A KWin script returns nothing to its caller; console.log lines land in the `plasma-kwin_wayland.service` user journal, which `arrange-to-tiles.sh` reads with `journalctl --user -u` and `--since`.
- `kscreen-doctor -j` carries `priority` and `scale` per output; on Neve DP-2 is priority 1 at scale 1.5 and eDP-1 priority 2 at 1.25.
  `kscreen-doctor -o` on Neve gives DP-2 as 3840x2160 with logical geometry 2560x1440, and eDP-1 as 1920x1200 with logical geometry 1536x960.
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
- These KWin facts are unobserved, since the parent declined to have them probed during this discussion:
  - `rootTile(output, desktop)` with `split`, `relativeGeometry` and `manage` on a desktop that is not current, which D3's apply and D6's grid rest on;
  - setting a window's `desktops`, and `workspace.sendClientToScreen` for a window on another output, since a window's `output` is read-only, which D3's move and D4's tiling screen rest on;
  - setting `frameGeometry`, and reading `clientArea` and a tile's `absoluteGeometry`, `padding` and `windows` on a desktop that is not current, which D6's clamp and zone reading rest on;
  - `kscreen-doctor -j` output names against `workspace.screens` names, which D4's match rests on;
  - `rootTile` returning two different root tiles for one output and two desktops, which D1's per-desktop tiling probe rests on, since QJSEngine ignores a surplus argument and an arity check proves nothing;
  - a script's body running to its end once `run` is called, whatever happens to the caller, which D3's end-of-run guarantee rests on;
  - `isScriptLoaded` reading false once an unloaded script has left KWin's list, which D3's script lock rests on;
  - a running Konsole window's resource class reading `org.kde.konsole`, which D2's match rests on.

  The implementing run's first step probes them, each with a KWin script run through `qdbus6` and its console.log lines read from the journal.
  It may run the read-only probes itself, where its session reaches the user session bus: the output names, the two root tiles, a running Konsole window's resource class, and a script unloaded right after `run` that still logs its end marker and then reads as not loaded.
  Where its session cannot reach that bus, it asks its parent to have the operator run them.
  A probe that moves a window or builds a grid, on a throwaway window and a desktop that is not current, runs only when the operator runs or approves it.
  A fact a probe contradicts stops the implementing run before any code relies on it: the run sends the probe's command and output to its parent, which reopens the decisions that rest on the fact.

<!-- lym:carry-over Discussion-Review -->
- Discussion-Review, round 3: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lym/reviews/discussion/round-3-review.md; fixer report: _lym/reviews/discussion/round-3-fixer-report.md.
  - konsole-resource-class-wrong-on-wayland (class design, severity BLOCKING, open in rounds 3)
  - query-report-lacks-what-ordering-needs (class design, severity BLOCKING, open in rounds 3)
  - renumber-rewrites-unsettled-window-keys (class design, severity BLOCKING, open in rounds 3)
  - script-run-reaches-another-script (class design, severity BLOCKING, open in rounds 3)
<!-- /lym:carry-over Discussion-Review -->

## Acceptance criteria

- D1: with `KDE_SESSION_VERSION` unset, another desktop named, an X11 session, a missing tool or a launcher other than Konsole, the Konsole command line is today's and no KWin call is made; a placement error is a logged warning and the verb exits zero.
- D1: under mode `off` with every check passing, the window's place is recorded and no apply runs; a query report whose probe fails leaves the `--separate` window where KWin opened it and settles its entry, with a warning.
- D2: with the pre-launch checks passing the Konsole command line carries `--separate` and the launcher returns the process id; a snapshot window with the right `pid` and caption and the resource class `org.kde.konsole` matches, and one with the right `pid` but another resource class, `konsole` included, or another caption does not match.
- D3: over recorded `qdbus6`, `journalctl` and `kscreen-doctor` output, the readers parse the snapshot, with each window's `internalId`, the current desktop and each desktop's leaf tiles with their `absoluteGeometry`, `padding` and `windows`, and the apply report; a window's zone is the leaf that lists it, whatever its `tile`; a `tier0` test runs them over every recording.
- D3: two runs that overlap, a window-wait query and another child's apply, hold the script lock one after the other, each under its own plugin name and each reading its own end marker; a load that returns -1 fails the run; a plugin name a killed run left in `kwinscript.loaded` is unloaded before the next load; a run whose plugin name still reads as loaded after its 2-second wait releases the script lock and leaves the name in `kwinscript.loaded`, and the next run fails before its load while that name stays loaded; an acquire abandoned past its bound that takes the lock later releases it at once; a window-wait child that finds a query report newer than its previous query reads it and loads no script.
- D3: lines of another run id are ignored, and a journal with no end marker within the bound fails the run as a timeout and unloads its script by plugin name.
- D3: the apply script addresses windows by `internalId` and sets the placed window's desktops to the target alone and sends it to the tiling output.
- D3: the planner, given a snapshot, rules and places, returns the target desktop, grid and window order; tier-1 tests cover each rule in D4 to D9 through it.
- D4: the output with the lowest priority among enabled outputs is the tiling screen, and windows on other outputs are not counted.
- D5: a 2560x1440 logical output gives capacity 2x5 and a 1536x960 one gives 1x3 with the template sizes; a tiny minimum still caps at 2x5; a desktop holding capacity minus one windows plus the placed window is not full.
- D6: 3 windows with 5 columns give 1x3; 6 give 2x3; 7 give 2x4 with the last zone empty; windows with remembered keys 2, 0 and 1 reopening in that order end in key order.
- D6: on a 2-row grid whose bottom row runs under a panel, each bottom-row window is released and its frame ends at the usable area's bottom, and at the next placement it counts in its zone; on a 1-row grid under such a panel every window is clamped, in its zone's box and untiled; a window on two desktops is not counted.
- D7: a task `run_settings` value wins over its beam's, a beam's over the prime's, `""` falls to the machine key, and an unknown value is refused on load.
- D8: `next` from desktop 1 with desktop 2 full lands on 3; past the last desktop nothing is moved and the warning names the start desktop; a home desktop wins over the mode; a parent whose placement settles its entry during the wait anchors its child on the desktop it was placed on.
- D8: a child whose parent's entry is unsettled under the lock waits again and anchors on the desktop the parent's placement settles it on, including a parent under mode `off` and a parent whose full home overflowed it; every return to the wait falls within one 180-second bound.
- D8: with no parent entry the child waits, and past the wait the current desktop is the anchor and the child's entry takes no home, so its next placement anchors on its parent; the wait's re-reads back off from one to eight seconds.
- D9: after an apply, the places file holds every reported window of the hub; a reopen that keeps the keys' order keeps the keys, a hand move renumbers the desktop; a worktree the hub no longer has is dropped; an absent file reads as empty.
- D9: a pre-wait write and a post-apply write that interleave keep each other's entries, and the pre-wait write keeps the home desktop, order key and last desktop.
- D9: another child's apply report showing a window whose entry is unsettled leaves that entry's home as it was; a child that ends on a timeout or a failed probe settles its entry; another child's apply on a different desktop leaves the order key of an unsettled window sitting on its home unchanged, and windows reopening on their home desktop end in key order.
- D9: a window that overflows from its full home keeps its home and order key, and its next placement with room returns it there; a renumber keeps an absent entry between the windows it sat between, and no two entries share a key; a renumber of the desktop an overflowed window landed on leaves that window's home and order key unchanged, and a renumber of its home keeps its position there.
- D10: the opening verb returns before the window appears in KWin; two children started together run their KWin steps one after the other; one that cannot take the lock in time, or whose window never appears, logs a warning, runs `window_command` and exits; a child that returns to the parent wait shares one 120-second lock timer across its acquires.
- D10: a child waiting on a held lock makes no attempt until the lock frees; the window wait's query runs start at least one second after the previous run ends.
- D11: a placement touches no desktop but its target, and a desktop at or over capacity is never rebuilt.
- D12: `lym reed window` and `lym machine start` both reach the one placement call; with the pre-launch checks passing, the child runs `window_command` with no sleep after placement, under mode `off`, and after a lock timeout, a window timeout and a failed probe; a child ended by SIGTERM runs none and ends one already running.
- D12: with the pre-launch checks failing, `lym reed window` runs `window_command` after the wait as today and `lym machine start` runs none.
- Review: the desktop template's comments for the new keys name the machine-layer set command, and `PATTERN-config-layers`'s background lists `beam.window_placement` (new) among the per-task beam settings.
- Review: `lym reed window`'s Long help and the desktop template's `window_command` and `window_wait_ms` comments say that under placement the placement child runs `window_command`, with no wait, except when the child is ended by SIGTERM.

## Notes for the plan writer

- The operator's `~/.local/bin/set-tiling-grid.sh` and `arrange-to-tiles.sh` are working references: the first releases the tiled windows, removes the root tile's children, splits columns then rows and sets each tile's `relativeGeometry`.
  The second filters windows and calls `manage` per zone.
  It then releases each zone's window whose bottom edge runs under the panel, logs the clamped geometry, and moves and sizes that window with `kdotool` from the journal line.

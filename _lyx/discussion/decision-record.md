## Goal

A reed session's visible windows follow one fixed order: the run overview first, then one window per agent in start order, then the shell (Selvage) last.
Attaching and every automatic switch show the most relevant running agent, falling back to the overview and never landing on the shell.

## Scope

In:

- A new display anchor, `overview`, and the window order overview, agents, shell in every reed session.
- The loom-status strand (`lyx loom checklist`, both loom and darn runs) and the prime's `runs` strand (`lyx batten overview`) move to the `overview` anchor.
- The loom LLM driver strand and the orch's orchestrator strand move to their own windows.
- Selvage's window is named `shell` and kept last; its button replaces VIEW.
- Focus rules: attach, a new agent window, a viewed window ending.
- Alt+Up and Alt+Down step across windows.
- The loom checklist frame: the full row list, the wait reason in full, the per-row detail.
- Both overview panes drop content-size reporting and truncation.
- A shared self-re-exec helper, used by `lyx loom checklist` and `lyx batten overview` on a changed binary.
- `reapplyLayout` stops toggling zoom on a re-apply that changes nothing.
- Bringing a live session booted by an older `lyx` into the order.

Out:

- The hub watchdog's restart on a new binary: `logging-cleanup` (`a90ef61d1`) already retires it and spawns a successor.
  The lock gap the board worried about is harmless: a racing spawner that wins the lock makes the successor exit on a lost race.
- Restarting `lyx orch watch`, an operator's `lyx reed attach` and other long-lived verbs: board entry `stale-binary-daemons`, which builds on D8's helper.
- Retiring `AnchorTop`, `AnchorBelowParent`, content sizing or the stacked layout: they stay valid for an explicit caller and keep working in the shell window.
- Per-row detail in the batten overview: its rows are tasks whose line already carries the task's state and reason.

## Decisions

### D1. A new `overview` anchor places one strand in the first window

`render.AnchorOverview` (`"overview"`) joins the closed anchor vocabulary.
Its strand gets a window of its own named `overview`, opened before the session's lowest-index window (`new-window -d -b -t <lowest window>`), so it holds the lowest index whenever it is opened.
At most one `overview` strand exists per session: adding a second under another name is refused, naming the existing strand and `lyx reed remove` of it as the way forward.
`ReplaceStrand` kills the old pane before launching the new one, which ends a one-pane window, so a replace of the overview strand, including one from another anchor (D10), opens a fresh overview window, again first.
When the replaced strand's window was the session's current window, the replacement's window is selected.
The overview strands set no segment, so the overview button takes the plain window-button style.
Rationale: the overview is a placement, so it belongs in the anchor vocabulary callers already choose from; one per session keeps "the first window is the overview" true.
Bound: the anchor places a single strand per session, only in the first window.

### D2. Window placement

- Own-window strands, the overview excepted, open with `new-window -d -b -t <shell window id>`: after every earlier agent window, before the shell.
  A relaunch keeps its window; a replace opens a new window before the shell, so it counts as the newest.
- `OpenWindow`'s hidden windows open after the highest-index window (`new-window -d -a`), so all of them sit after the shell, and the shell is the last visible window.
- When no shell window resolves (no Selvage pane recorded or alive), a window opens with the plain `new-window -d`, logged at Debug.
- On psmux, a refused `-b` or `-a` falls back to the plain form, logged at Debug: the order is best-effort on Windows and nothing fails.

Rationale: inserting before the shell keeps start order with no sort pass and no reserved index, so window index order is start order.

### D3. Selvage's window is the `shell` window

The session's first window, where boot creates Selvage, is renamed `shell` at boot with automatic renaming off.
It stays the strands' window that `strandWindowTarget` resolves through the Selvage pane, so a strand an explicit caller anchors `top` or `below-parent` still lays out there above Selvage, and the stacked layout, zoom bracket and resize loop keep working for it.
Rationale: moving everything else out leaves Selvage alone in that window with no change to how reed finds or lays it out.

### D4. The driver and orchestrator strands get their own windows

The loom LLM driver strand (`loomcli/driverspec.go`) and the orch's orchestrator strand (`orchcli/start.go`) change from `below-parent` to `own-window`.
The orchestrator keeps `Focus: true`, which for an own-window strand selects its window once opened.
Rationale: in the prime, the batten overview takes the first window, and the operator asked for every agent in a window after the overview and for the shell alone last.

### D5. Focus rules

An agent window is a window holding an own-window strand; the overview, shell and hidden windows are not agent windows.
The relevant window is the highest-index agent window with a live pane, which by D2 is the newest live agent, else the overview window.

- **Attach.** `AttachArgv`'s chain selects the relevant window; with neither an agent nor an overview it leaves the current window.
- **A new agent window.** Reed selects it only when the session's current window is the overview or an agent window whose panes are all dead; otherwise it keeps `@lyx_unseen`.
  The shell counts as watched.
  A strand with `Focus: true` is selected regardless.
- **A viewed window ends.** When `RemoveStrand`, `ReplaceStrand` or reconcile ends the session's current window, reed first selects the relevant window among those left, never the shell; with none left it leaves the choice to tmux.
  A strand leaving a shared window that stays open switches nothing.
  This replaces `selectStrandsWindowIfCurrentLocked`'s choice of the strands' window; D1's replace rule takes precedence for the overview.

Each rule reads reed state, window ids and pane liveness, never `@lyx_unseen`, `@lyx_hidden` or a wait mark (`PATTERN-wait-mark-display-only`).
Rationale: a run spawns its current row's agent last, so the newest live agent is that row's in the common case, and reed reads no run semantics.
Bound: an automatic switch happens only at these three points, never moves a viewer off the shell or a live agent window, and races an operator's own window change by at most one switch.

### D6. Navigation and the status bar

Alt+Down runs `next-window` and, when it lands on a window carrying `@lyx_hidden`, selects the session's first window (`:^`).
Alt+Up runs `previous-window` and, when it lands on a hidden window, selects the window carrying `@lyx_strands`, the shell.
Both wrap and work in every window; the `@lyx_strands` gate on the keys goes.
Since hidden windows only sit after the shell (D2), one conditional step always reaches a visible window.
Stepping between panes of the shell window is dropped; a click on a strand's pane button still reaches it.
The window buttons on line 1 follow window index order, so the overview's comes first and the shell's last.
The `shell` button replaces VIEW in the click dispatch and keeps VIEW's action: select the strands' window and unzoom it.
Alt+z keeps zooming only in the strands' window.
Rationale: windows are the unit the operator moves between, in the order D1 to D3 fix.

### D7. The overview panes' frames

Both overview panes, `lyx loom checklist` and `lyx batten overview`, run without a content-size report and without a truncating fit: in a window of their own reed sizes nothing, and every row is drawn.
The checklist frame is, top to bottom: a detail block per visited row in recipe order, then the row list (box, time spent, visits, which for a review row are its rounds), then the state line and the wait line.
A terminal shows the last lines of a frame taller than itself, so the list and the wait stay in view, and the operator scrolls up (copy mode, the mouse wheel) for the detail.
A row's detail lists each of its visits from the shed status history: when it ran, how long (derived from the next entry, as the visit times are today), its outcome and output, and its gate attempts; no new persisted field.
The wait line shows the full reason `lyx loom status` gives; rewording the `reviewWaitingFor` note alone does not meet this.
A wait on an escalation to the parent says so in words, for instance "waiting on the parent: escalation from Plan-Bouncer".
The batten overview's frame is otherwise unchanged.

### D8. A shared self-re-exec helper

A new leaf package, `internal/selfexec`, importing only the standard library and `internal/logger`, holds the helper.
At start it records the path `os.Executable` resolves and that file's identity (device, inode, size, modification time); each check re-stats the recorded path.
When the identity differs from the start and is the same on two consecutive checks, it logs at Info that the process replaces itself, then `syscall.Exec`s the recorded path with the same argv and environment.
A failed stat counts as unchanged and logs at Debug.
A failed exec logs at Warn and is not retried for that identity; a later, different identity is tried again.
It never execs under `go test`.
On Windows, which has no exec, the check reports unchanged and the process keeps running; a Windows pane restarts through `lyx loom start` or `lyx orch start`, as today.
`statuspane.Pane.Run` calls the check on each poll tick, adding no wake of its own (`PATTERN-daemon-wakeups`), so both overview panes re-exec in place and keep their pane, strand binding and `TMUX_PANE`.
`stale-binary-daemons` can call the same helper from its long-lived verbs; the watchdog keeps its retire-and-respawn path, which hands the lock over.
Rationale: exec keeps the pane in place where a respawned process would leave a dead pane, and one package gives the next callers the same rule.
Bound: the exec runs only the recorded path with the process's own argv, so it can start nothing the process was not already running.

### D9. The resize re-apply toggles zoom only for a new box

`reapplyLayout` resolves the live box before entering `keepZoomLocked`.
It skips the apply, and the bracket, when the box equals the last applied box or the box whose apply last failed, and enters the bracket only for an apply it will run.
A strand change still re-applies through its own op, so a failed box is retried there or when the box changes.
Rationale: a re-apply that changes nothing, failed or not, must not unzoom and re-zoom the window, the flicker the board entry traced.

### D10. Live sessions from an older `lyx`

Before every window open and in the attach pre-flight, reed checks the strands' window: it renames it `shell` when it is not, and moves it after the last non-hidden window (`move-window -a`) when a visible window sits after it.
Strands already placed stay where they are: `Resume` and a revive relaunch a strand from its persisted display, so an in-flight run's `below-parent` LLM driver stays in the shell window until the run ends.
A strand placed from a fresh spec takes its new anchor: `lyx loom start` replaces a status strand launched by an older build (`statuspane.StrandCurrent`), which moves it to the overview window by D1, and a new driver spawn opens its own window.
Rationale: no pane is moved under a live agent, and the check costs one rename and one move once per old session.

## Constraints

- `PATTERN-wait-mark-display-only` (D5), `PATTERN-spawn-observability` and `PATTERN-daemon-wakeups` (D8) bind as the Decisions cite them.
- `internal/selfexec` documents its closed import set in its `doc.go`; `internal/statuspane/doc.go`'s import list gains it.
- `PATTERN-log-levels`: the Debug and Warn levels in D2, D8 and D9 follow its policy; nothing logs per poll above Debug.
- `PATTERN-test-speed` and `PATTERN-test-isolation`: window order and focus are tested at the tmux tier through `tmuxkit.Main`, the rest at the lowest fitting tier.
- Docs land in the same commit: `internal/reedengine/doc.go` (window order, anchors, focus, bindings, bar), `internal/statuspane/doc.go`, `internal/selfexec/doc.go`, the loomcli, orchcli and battencli docs that describe their strands and panes, and `docs/overview.md` where it describes the run window.

## Auto-mode assumptions

- D5's newest live agent stands in for the board's "the run's current row's agent".
- D5 counts the shell as watched, beyond the board's "another agent".
- D4 applies the order to the prime's orch session.
- D7 reads row detail from the existing status history only.
- D8 gives Windows no re-exec.
- D6 drops pane stepping in the shell window.

## Open risks

- In the prime the operator sees the batten overview or the orchestrator, no longer both at once; `orch-split` will add a second agent there and decides which of its agents carries `Focus`.
- A new build that exits at once on the same argv (a renamed hidden verb, a startup crash) leaves the overview pane dead until `lyx loom start` or `lyx orch start` replaces it.
- tmux copy mode shows a frozen copy of the frame, so a redraw while the operator scrolls appears only when copy mode is left.
- A row list taller than the window scrolls its earliest rows off the top; the state and wait lines stay in view.

## Acceptance criteria

- In a session with a run, window indexes read overview first, agent windows in start order, shell last among visible windows; this holds after an agent window ends and a new one opens, and after the overview strand is replaced while agents are live.
- Replacing a `top` loom-status strand with an `overview` strand opens the overview window first and removes the old pane from the shell window; replacing the strand of the viewed overview selects the new overview window.
- Two hidden `OpenWindow` windows open after the shell, and Alt+Up and Alt+Down never stop on either.
- A second `overview` strand under another name is refused with a message naming the existing strand.
- Attach selects the newest live agent's window; with no live agent the overview; with neither it changes nothing.
- A new agent window is selected when the current window is the overview or a dead agent window, and not when it is a live agent window or the shell, where it carries `@lyx_unseen`; a `Focus: true` strand is always selected.
- Ending the viewed window, by removal, replace or reconcile, selects the newest remaining live agent's window, else the overview, never the shell; one seat leaving a shared window that stays open switches nothing.
- Alt+Up and Alt+Down step across visible windows in index order and wrap.
- The status bar shows window buttons in index order with `shell` last and no VIEW; clicking `shell` selects the strands' window and unzooms it.
- `lyx loom start` puts the loom-status strand in the overview window and an LLM driver in its own window; `lyx orch start` puts the runs strand in the overview window and the orchestrator in its own, selected window.
- Neither overview pane calls `SetContentRows` or draws a "more rows not shown" line.
- The checklist frame ends with the row list, state line and wait line, holds a detail block per visited row above the list, and words an escalation wait as waiting on the parent.
- The re-exec helper execs the recorded path with the process's argv once a changed identity holds over two checks; a failed stat never execs; a failed exec keeps the process running, logs once and is not retried for that identity; nothing execs under `go test`.
- `reapplyLayout` with the last applied box, or with the box whose apply last failed, issues no `resize-pane -Z`.
- In a session booted by an older `lyx`, the next attach or window open renames the strands' window `shell` and moves it last, and a resumed `below-parent` strand stays in the shell window.

## Notes for the plan writer

- Own-window opening is `spawn.go`'s `openOwnWindowLocked`; detached windows are `window.go`'s `OpenWindow`; the end-of-window switch is `ownwindow.go`'s `selectStrandsWindowIfCurrentLocked`.
- The status bar and click dispatch are `bar.go` and `bindings.go`; the step and click commands are pure builders with unit tests beside them.
- The watchdog's `exeWatch` in `internal/reedcli/watchdog.go` is a model for the identity check.
- Check `cmd/lyx/testdata/test-budget.yaml` for `internal/reedengine`, `internal/statuspane`, `internal/loomcli` and the new package before adding top-level tests.

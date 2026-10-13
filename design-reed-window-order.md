# reed: overview window first, agents next, shell last

`reed-window-order` · Task · bug, reed, loom, shed · running · PR-Rework

## Seen (operator, 2026-10-13)

Since `reed-multi-window` each strand gets its own tmux window, but the session's window 0 holds Selvage (and an LLM driver strand, with the loom status pane above it), and `lyx reed attach` lands there.
A strand such as `discussion` opens in window 1, detached and never selected (`internal/reedengine/doc.go`, "Detached windows" and the own-window strand lines), so the operator attaching to a run does not see the agent at work and has to switch windows.

Alt+Up and Alt+Down do not reach the other windows: they step between the panes of the strands' window only (`internal/reedengine/bindings.go:71-72`, `select-pane -t :.-`/`:.+`, gated on `@lyx_strands`).

The status bar's VIEW button selects the strands' window and unzooms it (`internal/reedengine/bindings.go:33-36`, `bar.go:27-28`); with the strands in their own windows that window holds little but Selvage, so its name misleads.

The loom status pane (`lyx loom checklist`) shares window 0 at about five lines, so it shows three rows and "… 23 more rows not shown".
Its wait line says only "wait bouncer Plan-Bouncer" while the run waits on a circling escalation to the parent; `lyx loom status` holds the full reason.
A pane started before `run-window-bugs` landed (`e17b2921`, the status pane clears screen and scrollback on redraw) keeps the old binary and stacks earlier renders in its scrollback until it restarts.

Window order today is tmux's creation order: every window takes the next free index (`new-window -t '=<session>:'`, `internal/reedengine/spawn.go:293`), and the window buttons follow index order; no reed code sorts.
Focus today: a new strand window opens with `new-window -d` and never takes focus; its button gets the unseen mark (`@lyx_unseen`, underlined) until selected.
The one automatic switch: when the window being viewed ends, `selectStrandsWindowIfCurrentLocked` (`ownwindow.go`) selects the strands' window.

## Wanted layout (operator, 2026-10-13)

Windows give more room than panes in one window, so:

- Window 0 is always the overview: a loom or shed status in a window of its own, for any kind of shed run (an LLM-driven or Go-driven loom, darn, batten), showing exactly how far the run has come, with the detail of each row reachable by scrolling on.
- The agents a run spawns, the LLM driver included, each get a window after the overview.
- The last window is always the shell: Selvage, kept one per session, since it keeps the session alive when every loom pane has gone and is always a terminal into the worktree; its button is named `shell`, in lower case like the other window buttons, and replaces VIEW.
- What matters most to the operator is seeing the most relevant running agent at any time.

## Scope to discuss

- Which window attach and automatic switching show: the overview, or the most relevant running agent (the run's current row's agent, else the newest); a new agent window takes focus only when the operator is not watching another agent.
- When the viewed window ends, switch to the next relevant agent window, else the overview, never the shell.
- Keeping the order fixed: the overview at index 0, the shell at a fixed high index (for instance `move-window -t :99`), agents between in start order.
- Alt+Up and Alt+Down step across every window in that order.
- The overview's content: the full row list with time and rounds, the wait reason in full (an escalation waiting on the parent named as such), and how the operator scrolls to a row's detail.
- The status pane restarting on the new binary after a deploy, so a fix reaches runs already in flight.
- `logging-cleanup` (landing soon) changes `reapplyLayout` and the watchdog's resize path; check at Discussion what it already fixed and drop that from scope.

## Zoom flicker (2026-10-13)

Cause: the hub's resize watchdog (pid 3854, started 2026-10-08, exe deleted so it runs the old binary) is in poll mode and calls `reapplyLayout` every 2 s per session.
`reapplyLayout` wraps its whole body in `keepZoomLocked` (`internal/reedengine/reapply.go:130`), and `keepZoomLocked` unzooms before running the step and re-zooms after it (`internal/reedengine/zoom.go:57-76`), even when the step is a no-op or fails.
So a zoomed strands' window is unzoomed and re-zoomed every poll cycle, which is the flicker.
Evidence: `_board/.lyx/logs/trace-20261008T181526Z-f2e3f7df172f3d30-3854.log` holds about 29000 lines of `reed: resize re-apply failed`, one per session every 2 s (`session=logging-cleanup` last at 10:07:04, then the sink hit its size cap); the error is `plan layout: render: strand ... uses deferred anchor "own-window"`, which the old binary raises for any session holding an own window (both sessions have one: `burler-review-chair`, `webster`).
A failed apply counts as "no change" only in current source (`watchloop.go:204-208`), and the old binary kept the 2 s cadence, so it never backed off.
Ruled out: the status pane redraws only when its text changes (`internal/statuspane/pane.go:96-103`; capture was stable over several samples) and reports rows only on a count change; the status bar interval is 15 s (`status-interval`); `reed-resize.signal` mtimes (11:16 and 11:20) show no periodic window resize, so the `window-resized` hook is not firing.
Own windows with no zoom avoid it: `keepZoomLocked` reads the strands' window only, finds `window_zoomed_flag` 0 and runs the step plain, so nothing is toggled; a window the watchdog never touches is never unzoomed.
Fix direction: restart the watchdog on the new binary after a deploy (the board item above already asks the same of the status pane), and make `reapplyLayout` read the live box before `keepZoomLocked`, so a skipped or failing apply never unzooms.

## Folded in from stale-binary-daemons (2026-10-13)

Long-lived reed-side processes re-exec themselves on a changed binary, through one shared helper:

- The loom status pane (`lyx loom checklist`) and batten's overview re-exec on a changed binary, so a fix reaches runs already in flight.
- The single-instance `lyx reed watchdog` re-execs with its lock handed over across the exec (keep the flock fd), so no second instance starts in the gap; the old watchdog on a deleted binary caused the zoom flicker.
- The build identity to compare against exists already (`internal/shedverbs/steprecord.go`, `binary_changed`).
- Windows has no exec; what happens there is a point to discuss.

Evidence carried over:

- The hub's `lyx reed watchdog` (single instance per hub, spawned by `up`, `resume` and `attach` only when no daemon holds the lock) ran from 2026-10-08 on a deleted binary.
  It re-applied the layout of every session every 2 s and failed each time with an error only the old binary raises (`deferred anchor "own-window"`), and each attempt unzoomed and re-zoomed the strands' window (`internal/reedengine/reapply.go:130`, `zoom.go:57-76`).
  It also filled its trace log to the size cap (`_board/.lyx/logs/trace-20261008T181526Z-f2e3f7df172f3d30-3854.log`); the orch killed it and started a new one by hand.
- After the kill, a run's `lyx loom checklist` status pane still ran a deleted binary (`/proc/<pid>/exe` marked deleted), started before the `run-window-bugs` deploy, so it lacks the status-pane fix.
- Shed already records a build identity per step and reports `binary_changed` (`internal/shedverbs/steprecord.go`).

# reed: give each strand its own tmux window, add lyx shuttle open

Every agent a run spawns now gets a full-size tmux window instead of a pane squeezed into the strands' window.
The windows stack in depth like tabs in the run's terminal window.
Window 0 keeps the driver, its `loom-status` strand, Selvage and every strand that asks for window 0.
A new window never takes focus.

## What changes

- `reed`: the `own-window` anchor, refused until now, is a supported placement.
  It covers add, launch, liveness, reconcile, removal, resume after a server restart, the status bar and the watchdog's name repair.
  `lyx reed add` and `lyx shuttle run` accept `--anchor own-window`.
- `reed`: strands can share a window per group.
  `render.Display` gains `Window` (the group name) and `WindowSize` (the strands the spawner puts in the group), both persisted with `omitempty`.
  `reed.yaml` gains `window_share_max` (default 2) and `window_split` (`width`, the default, or `height`).
- `shuttle`: a run's strand defaults to `own-window`, in run and attach alike.
  The loom driver and the orch name `below-parent` explicitly and stay in window 0.
- `seatengine`: a table's seats declare one group sized by the seat count, so a two-seat table shares a window and a five-seat table gives each seat its own.
  Two rounds of one table never share a window.
- `shuttle`: new verb `lyx shuttle open --role <role> [--model] [--segment]` opens a named LLM session with no prompt, no run record and no output files, in its own window, with its session id recorded so the watchdog's name repair covers it.
  It replaces the need for the board's `extra-agent-strand`.
- `shell` gains `OrElse` and `Exit`, and `claudeengine` gains the no-prompt named-session launch and its resume.
- Docs: the affected `doc.go` files are updated, and the operator command index gains `lyx shuttle open`.

## Design decisions

- Shared or apart is decided from the declared `WindowSize` against `window_share_max`.
  The first strand of a group records the choice, and later joiners follow it, so a `reed.yaml` edit mid-group never splits a group.
  A joiner that would push a shared group past its recorded size gets a window to itself.
  A join declaring a different size than the live group's is refused.
- `window_split` and `window_share_max` are validated at up, resume, add and replace.
  Remove never fails on a bad value: it removes the strand and reports the invalid key, so a `reed.yaml` typo cannot block cleanup.
- Window 0 stays defined by Selvage.
  Liveness, reconcile and name repair read own-window panes through a session-wide listing.
  Reconcile's untracked-pane reap, dead-pane kill and survival rule stay limited to window 0.
- A dead agent in an own window keeps its dead pane, and its strand binding, until the strand is removed.
  Removal first moves a client viewing that window to window 0, then ends the window.
  Shuttle's stop paths (the stop verb, step stop, supersession on respawn, a seat table's stop) remove a present-but-dead own-window strand too.
  `Wait` on a run that ended `OutcomeDied` saves the pane's last capture into the run directory and removes the strand.
  A timed-out run keeps its strand.
- The new window announces itself on the status bar: a button in the segment color, marked unseen until first selected.
  A `session-window-changed` hook clears the mark, and it is skipped where the multiplexer lacks `set-hook`.
  The marks are display only.
- A change of anchor into or out of `own-window`, or of `Window` or `WindowSize`, is refused at `UpdateStrand`: remove the strand and add it again.
  Attach to a strand an older binary recorded in window 0 leaves it there.
- `lyx shuttle open` chains, after the provider exits, an exit marker write and reed's detached self-removal, so the window closes when the operator ends the session.
  If the self-removal fails, the dead pane stays until `lyx reed remove` or a pair's teardown, and `Resume` drops the strand through its done-when path.
  A pair's teardown ends the reed session first, so it never triggers the self-removal.
  It is refused while a pause covers the worktree.
- A relaunch of an own-window strand kills nothing.
  The old dead pane is left unbound, and reconcile ends its window after moving any viewing client to window 0, once no pane in it is bound.

Not in this change: cross-worktree columns, per-window attach through session groups, new window-stepping key bindings, moving a live strand between windows, and any change to zoom (it acts on window 0 only).

## To check by hand

- A detached window created while no client is attached takes tmux's default size until a client views it, so an agent starting there renders at that size.
  The shuttle readiness probe fix in `run-window-bugs` owns any fix and re-checks it against this layout.
- A fresh shuttle run with no attached client: confirm the agent's window, its unseen mark on the status bar, and that focus did not move.
- Relaunch of a strand whose own-window pane is dead (`internal/reedengine/spawn.go`, `launchStrandLocked`): confirm the old dead window ends through reconcile and not through a kill at relaunch, and that a viewer on it lands on window 0.
- `lyx shuttle open --role explore` in the prime: confirm the window closes when the session exits, a second call gives `explore-2`, and `Resume` after a server restart reopens it under its session name.
- The new tmux-tier tests in `internal/reedengine` were compiled but not run during the build.
  Run `lyx gate test ./...` and `lyx gate test --tags integration ./...` before relying on them.
- `internal/shell/posix.go`, `internal/shell/pwsh.go` and `internal/shell/shell.go` were extended with `OrElse` and `Exit`; review those additions on both shells.
- An older hub watchdog daemon left running after a deploy lists window 0 panes only.
  It leaves own-window strands untouched, so no daemon restart is needed.

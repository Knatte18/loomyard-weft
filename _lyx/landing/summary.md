# reed: give each agent a full-size tmux window of its own

Every agent a run spawns now gets a detached tmux window of its own, stacked like tabs in the run's terminal window, instead of a pane squeezed into the strands' window.
Window 0 keeps the driver, its `loom-status` strand, Selvage and every strand that asks to stay there.
A new window never takes focus, so the operator is never moved automatically.
The same anchoring lets the orch open an extra named Claude session, such as one for exploring the repo, in its own window.

## What changes

- **reed**: the `own-window` anchor, refused until now, is a supported placement across add, launch, liveness, reconcile, removal, `Resume` after a server restart, the status bar and the watchdog's name repair.
  `lyx reed add` and `lyx shuttle run` accept `--anchor own-window`.
- **Shared windows**: `render.Display` gains `Window` (the group name) and `WindowSize` (the number of strands the spawner puts in the group).
  Strands of one group share a window when `WindowSize` is at most the new `reed.yaml` key `window_share_max` (default 2); otherwise each gets a window named by its role.
  The new key `window_split` (`width`, the default, or `height`) sets the split direction.
- **shuttle**: `own-window` is the default anchor for a fresh spawn, in run and attach alike.
  The loom driver and the orch name `below-parent` explicitly and stay in window 0.
  Attach to a strand an older binary recorded in window 0 leaves it where it is.
- **seatengine**: a table's seats declare one group sized by the seat count, so a small table shares a window and a large one gives each seat its own.
  A later round never joins an earlier round's window.
- **`lyx shuttle open --role <role> [--model <spec>] [--segment <segment>]`**: opens a named session with no prompt in its own window, records its session id on the strand and returns the strand's name and guid.
  It writes no run record and refuses under a pause covering the worktree.
  When the provider exits, a chained exit marker and reed's detached self-removal close the strand and its window.
  The verb is added to the operator command index.
- **Status bar**: each own window gets a button in its segment color, carrying its strands' wait marks and underscored until first selected.
  A session-scoped `session-window-changed` hook clears the unseen option; it is re-pinned by every attach pre-flight.

## Design decisions

- reed owns the mechanics and the spawner only states intent in `Display`; the shared-or-apart choice is recorded on the group by its first strand, so a `reed.yaml` edit mid-group never splits it.
- A join that would exceed the group's recorded `WindowSize` gets a window of its own; a join declaring a different size than the live group is refused.
- `UpdateStrand` refuses a move into or out of `own-window` and a change of group; the way forward is remove and add again.
- An own window closes when its last strand is removed, and a client viewing it is first moved to window 0.
  Dead own-window panes are kept (`remain-on-exit`) until their strand is removed, and reconcile's dead-pane kill, untracked-pane reap and survival rule stay limited to window 0.
  A window ends only when it carries `@lyx_own` and none of its panes is bound to a strand.
- Shuttle's stop paths remove a tracked own-window strand whose pane is dead as well as live.
  `Wait` on a run that ended `OutcomeDied` saves the pane's last capture into the run directory and removes the strand; a timed-out run keeps its strand.
- Remove never fails on an invalid `window_split` or `window_share_max`; it reports the key in its result.
  Up, add and resume fail naming the key.
- Every launch goes through the one strand launch chokepoint, so the pane-binary prelude, agent name and title lock apply as in window 0.
- The pane shell seam gained `OrElse` and `Exit` to compose the chained self-removal.
- No watchdog restart is needed after deploy: an older daemon lists window 0 panes only and leaves own-window strands untouched.
- Out of scope: cross-worktree columns, moving a live strand between windows, and zoom, which still acts on window 0 only.

## Check by hand

- The new tmux-tier tests were only compiled during the work, not run; run `lyx gate test --tags integration ./internal/reedengine/...` and the shuttle and seat packages.
- Findings left open by review, to check in the code:
  - Relaunching a launched own-window strand must not kill a pane the kill is not licensed to reach, because the stale pane listing predates the relaunch (`internal/reedengine`, relaunch path in `spawn.go`).
  - A relaunch of an own-window strand must not leave its replaced dead pane behind; the last commit on the branch addresses this, so confirm it.
- A detached window created while no client is attached takes tmux's default size until a client views it, and an agent starting there renders at that size.
  The shuttle readiness probe fix in `run-window-bugs` owns any change; re-check it against this layout.
- Open a live run in a terminal and confirm: windows open without taking focus, the button reads unseen until selected, a dead agent's window shows its dead pane, and removal falls back to window 0.
- Run `lyx shuttle open --role explore` in a worktree, exit the session, and confirm the window closes.

# reed: give each agent a tmux window of its own

Every agent a run spawns now gets a full-size tmux window, stacked like tabs in the run's terminal window, instead of a pane squeezed into the strands' window.
Window 0 keeps the driver, its `loom-status` strand, Selvage and every strand that does not ask for a window.
The orch can also open an extra named Claude session, such as one for exploring the repo, in its own window.

## What changes

- reed: the `own-window` anchor, refused until now, is a supported placement across add, launch, liveness, reconcile, removal, resume after a server restart, the status bar and the hub watchdog's name repair.
- reed: `render.Display` gains `Window` (a group name) and `WindowSize` (the group's strand count), both `omitempty`.
  Strands of one group share a window up to the new `reed.yaml` key `window_share_max` (default 2); a larger group gives each strand a window named by its role.
  The new key `window_split` (`width`, the default, or `height`) sets the split direction of a shared window.
- shuttle: a run's strand defaults to `own-window`, in run and attach alike, and `lyx shuttle run --anchor` defaults to it.
  The loom driver and the orch (`internal/orchcli`) name `below-parent` explicitly and stay in window 0.
- seatengine: a table's seats declare one group sized by the seat count, so a small table shares a window and a large one gives each seat its own.
- shuttle: new verb `lyx shuttle open --role <role> [--model <spec>] [--segment <segment>]` opens a named session with no prompt, no run record and no output files, and records the session id on the strand so the watchdog keeps the name across a compact.
  When the provider exits, a chained exit marker and reed's detached self-removal end the strand and its window.
  The verb refuses while a pause covers the worktree.
  It is added to the operator command index.
- `lyx reed add` and `lyx shuttle run` accept `own-window` for `--anchor`.

## Design decisions to review

- A new window never takes focus.
  It is created detached and carries `@lyx_own`, `@strand_color` and `@lyx_unseen`.
  The status bar shows a button in the segment color, underscored until first selected, cleared by a session-scoped `session-window-changed` hook.
  Where the multiplexer lacks `set-hook` (psmux), `@lyx_unseen` is never set.
- A group's shared-or-apart choice and `WindowSize` are recorded by its first strand, so a `window_share_max` edit mid-group does not split it.
  A join declaring a different size than the live group's is refused.
  A join past the recorded size gets a window to itself.
- Removing an own-window strand moves a client viewing that window to window 0 before the window ends; a client elsewhere does not move.
- An agent that dies in an own window leaves its dead pane showing until its strand is removed; reconcile's dead-pane kill, untracked-pane reap and survival rule stay limited to window 0.
  Shuttle's stop paths (stop verb, step stop, supersession on respawn, seat table stop) remove a tracked own-window strand even when its pane is dead.
  `Wait` finalizing a run `OutcomeDied` saves the pane's last capture into the run directory and removes the strand; `OutcomeTimeout` keeps it.
- `UpdateStrand` refuses a move into or out of `own-window`, and a change of `Window` or `WindowSize`; remove and re-add instead.
- Remove never fails on an invalid `window_share_max` or `window_split`; it reports the key in its result so a `reed.yaml` typo does not block shuttle's cleanup.
- `internal/shell` gains `OrElse` and `Exit` for the chained self-removal; provider specifics of the open launch live in `claudeengine`.
- Attach to a strand an older binary recorded in window 0 leaves it there.
- No daemon restart is needed after deploy: an older watchdog daemon re-applies window 0's layout only and leaves own-window strands untouched.

Out of scope: cross-worktree columns and tmux session groups (dropped by the operator), the shuttle readiness probe fix (`run-window-bugs`), new window-stepping key bindings, moving a live strand between windows, and zoom, which stays on window 0.

## To check by hand

- A window created while no client is attached takes tmux's default size until a client views it, and an agent starting there renders at that size; `run-window-bugs` owns any fix.
  Attach to a run and look at a window that was opened before the attach.
- `internal/reedengine`, relaunch: a relaunch kills the replaced own-window pane based on a pane listing that may be stale.
  Check that a relaunch of an own-window strand kills exactly the replaced pane and no other.
- The tmux-tier tests in `internal/reedengine/ownwindow_integration_test.go` were only compiled when written; confirm they pass on a machine with tmux (`lyx gate test --tags integration ./internal/reedengine`).
- Open a window, a shared pair and a five-strand group in a live run, and confirm focus stays put, the button shows as unseen, and removal falls back to window 0.
- Run `lyx shuttle open --role explore` and end the session; confirm the strand and window go away, and that the session name survives a compact.

# Clean pair lifecycle: no leftover sessions, branches, sockets or races

`pair-lifecycle` · Planned · feature

## Goal

Removing a pair ends its tmux session first, can finish a half-removed pair, and leaves no branches, sockets or stray directories behind; starting a run never races a dying driver.

## Also folded in, not in bolt before

- After refusal-audit landed, the driver wrote a drive report into the weft; `lyx fabric remove --remote` removed the task worktree, refused the dirty weft, and a rerun answered "worktree not found", so the half-removed pair had to be finished by hand.
- Warp branches on origin (see `fabric-branch-hygiene` below): `remove --remote` deletes the weft branch on origin but never the warp branch, and squash merges mean git never sees them as merged.
  On 2026-10-02 the hub deleted 37 such warp branches by hand (no open PR, no live pair).
  Want: `remove --remote` deletes the warp branch on origin too, and `fabric cleanup --remote` finds warp branches on origin with no live pair and no open PR.

## From `fabric-remove-ends-reed-session`: fabric remove leaves the pair tmux session alive

Seen 2026-09-30 for quarry-cli-answers and status-strand-by-driver: after landing and `fabric remove --remote`, their reed sessions were still alive hours later.
A driver that wrote its stop report after the removal recreated `<hub>/<slug>/.lyx/...` and `<hub>/<slug>/_lyx/shed/<slug>/drive-reports/<report>.md`: a stray directory, and a drive report that never reached the archive.
The hub killed the sessions and deleted the dirs by hand.

Fix: remove (and batten teardown) ends the pair reed session (reed down) before deleting the worktrees, and waits for or refuses on a live driver; a stop report written after done should land before the archive tag.

## From `driver-restart-race`: loom start races the ly-drive self-removal and ends with no driver

After a hand-back, the ly-drive session removes its own strand with a detached `lyx reed remove --name loom-driver --detach` that waits for the session's pid to exit (end-of-session command from `083e992cf`).
A `lyx loom start` issued in that window sees the loom-driver strand still live, keeps it instead of spawning a driver, and the detached remover then deletes it: the run is left with no driver and nothing happens (hit on `batten-hub-ready` and `focus-marker-warn`, 2026-09-30; a second `loom start` works).
Fix direction: `loom start` treats a driver strand with a pending self-removal (or whose claude has exited) as gone and spawns a fresh one, or the remover skips a strand that was re-adopted since it was scheduled.

## From `fabric-branch-hygiene`: Fabric: branches left behind by a failed add and on the remote

Read the closed GitHub issues #269 (add rollback leaves the warp branch) and #270 (cleanup cannot see weft branches stranded on the remote). Re-check both once `landing` has landed: it takes #293 (remove leaves the warp branch, locally and on origin), which touches the same branch-lifecycle code, and the ly-drive may repair the rest from a trace.

After `landing` (#298): `fabric remove` now deletes the local warp branch, but `--remote` still deletes only the weft branch on origin; warp branches of removed pairs stay there (`board-done-on-landing`, `burler-exclude-warn`, `integration-verify-fence`, `loom-start-layout`, `stencil-drift-remedy`, `stuck-reason-in-status`, `landing`). Out of scope in landing's decision record.

## From `main-weft-not-pushed`: main-weft is never pushed and never advances on landing

Seen 2026-09-30, uninvestigated: the prime weft branch main-weft stays local and does not move when a task lands

## From `test-tmux-socket-leak`: Integration tests leave tmux sockets behind

Tests leak tmux sockets: `/tmp/tmux-1000/` holds hundreds of `lyx-Test…`, `lyx-contract-…` and `lyx-warp-bare-…` sockets left by reed/loom integration tests. Test fixtures should kill their tmux server and remove the socket in `t.Cleanup`.

# Fixer report — discussion, round 1

## Changes

- F1 (BLOCKING), fixed with the review's option (a).
  The premise holds: `planReconcile` in `internal/reedengine/reconcile.go` puts every dead pane but Selvage's and the kept one into `deadPanesToKill`, and `launchStrandLocked` reconciles on every launch.
  D5 drops the false "exactly as its dead pane stays in window 0 today" clause and states that reconcile's dead-pane kill does not reach own windows, so a dead own-window pane keeps its binding until its strand's removal and the client select always runs first; window 0's dead-pane kill is unchanged.
  D6 limits the untracked reap, the dead-pane kill and the keep-one-dead-pane survival rule to window 0's panes.
  The dead-agent acceptance line now covers later reed ops in the worktree and the unchanged window 0 kill.
- F2 (MEDIUM), fixed as proposed: D12's Bound states that the pair's teardown ends opened sessions in a pair worktree, and that in the prime an opened session lives until its provider exits or `lyx reed remove`, comes back through `Resume`, and is accepted with no cap.
- F3 (LOW), fixed as proposed: D3 makes an invalid `window_split` or a negative `window_share_max` fail up, add, remove and resume naming the key, and gives `window_share_max` its range (an integer at least 0, 0 meaning every grouped strand gets its own window); the acceptance line matches.
- F4 (LOW), fixed with the review's second alternative: D2 records the shared-or-apart choice on the group by its first strand, and an over-count join to a shared group gets a window to itself instead of being refused, so a re-stepped seat is not refused; a new acceptance line covers the over-count join and the mid-group config edit.
- F5 (LOW), fixed as proposed after confirming the premises in `reapply.go` (persists nothing), `namerepair.go` (skips an absent or dead pane) and `reedcli/watchdog.go` (calls only `Watch`, `WatchNames` and `ReapSession`); the `hubreconcile.Ensure` clause is gone.
- F6 (LOW), fixed as proposed: the `lyx shuttle open` acceptance line gains the restart `Resume` and the pair teardown clauses.
- `support-log.md` Q10 and Q12 are brought in line with the F5 and F1 changes.

## Disputed

None.

## Deferred

None.

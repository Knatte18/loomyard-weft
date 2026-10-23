# Orch watcher: wait out an in-flight /compact before reloading

The orch watcher no longer lets a second `/compact` reach the orch session after a compaction it asked for has landed.
Before, such a `/compact` interrupted the resumed session's first turn, and the reload's color step waited behind it in creel's queue, leaving `lyx orch status` at `phase: resuming`.

The cause is in the compacting phase.
When it finds a qualifying compaction boundary it withdraws its still-queued `/compact` and dropped the pending step after any successful withdraw call.
Creel's withdraw removes only injects in its queue and leaves one a delivery attempt holds.
If the session refuses that attempt as busy, creel puts the inject back in the queue and types it at the next idle, ahead of every later watcher inject.

The fix is in `internal/orchengine/watcher.go`.
After the withdraw, `withdrawQueuedCompact` reads the pending inject's outcome again.
A withdrawn, injected or unconfirmed outcome lets the phase complete.
An outcome that still reads queued or sending keeps the step pending and the boundary unconsumed, and the next tick withdraws again.
The new `errCompactInFlight` signals this hold to `tickCompacting`.

The same commit also fixes the reed pane-tail separator.
tmux 3.4 escapes control characters in `-F` output, so the `\x1f` separator arrived as the literal text `\037` and the pane tail never split.
`paneTailSeparator` in `internal/reedengine/parse.go` is now a tab, which tmux passes through verbatim, and the `-F` formats in `overlay.go` and `ownwindow.go`, the parse comment and the `doc.go` format line match.
Before this, the `reedcli` and `reedengine` tests failed on `main`.

## Design decisions

- The hold lasts while one creel delivery attempt runs, which settles as typed or back in the queue.
  An inject creel types in that attempt cannot be stopped from the watcher, so the phase completes on it as on any typed `/compact`, and the session still compacts a second time in that case.
  Only creel could prevent it.
- A pending `/compact` with no id, which creel never answered and the journal lookup could not adopt, has no outcome to read.
  It is withdrawn once and dropped, as before.
- The hold is bounded.
  An inject still queued or sending past the handoff timeout from the phase entry is dropped with a warning and the phase completes, so a stuck delivery cannot hold the reload indefinitely.
- A failed read of the outcome after the withdraw is held as a creel failure, and a resume line the withdraw removed stays recorded in `State.ResumeLineWithdrawn`.
  A later call then clears the pause record's noticed mark although its own withdraw finds nothing.
- The wait repeats each tick, so it logs at Debug.
- Creel's delivery worker and withdraw verb, shuttle's send path and the reload step order are unchanged.

## Tests

`TestCompact_BoundaryWithdrawsAStillQueuedCompact` is extended and the creel fake models the withdraw as creel does: queued injects become withdrawn, in-flight ones stay.
It covers a boundary found while the `/compact` reads sending, then read as injected on a later tick, and read as back in the queue on a later tick.
It also covers a failed re-read after the withdraw and the handoff-timeout drop.
The `reedengine` tests are updated for the tab separator.

## Docs

The compact-cycle paragraph and the withdraw bullet in `internal/orchengine/doc.go`, the `State.ResumeLineWithdrawn` comment and the pane-listing format in `internal/reedengine/doc.go` describe the new behavior.

## To check by hand

- Not run: a real orch compact cycle against a live creel, where a delivery attempt holds the `/compact` as the boundary is found.

# Orch watcher: hold the boundary while a /compact is in flight

An operator-requested compact landed and the resume prompt reached the orch session, yet the watcher typed a second `/compact` into the resumed, busy turn and interrupted it.
`lyx orch status` kept reading `phase: resuming` with the reload's color step still pending.

## Cause

When the compacting phase finds a qualifying compaction boundary, it withdraws its own still-queued `/compact` from creel and drops the pending step.
Creel's withdraw removes only injects in its queue and leaves one a delivery attempt holds.
If that attempt is refused as busy, creel puts the inject back in the queue and types it at the next idle, ahead of every later watcher inject.
The watcher had already dropped the step after any successful withdraw call, so the stale `/compact` survived and the reload's color step waited behind it.

## Change

`withdrawQueuedCompact` in `internal/orchengine/watcher.go` now reads the pending inject's outcome again after the withdraw:

- A withdrawn, injected or unconfirmed outcome drops the step and lets the phase complete, as before.
- An outcome still queued or sending keeps the step pending and the boundary unconsumed, and returns `errCompactInFlight`.
  `tickCompacting` treats it like a held withdraw, and the next tick withdraws again.
- Past the handoff timeout from the phase entry, an in-flight step is dropped with a warning and the phase completes, so the wait cannot last forever.
- A failed read after the withdraw is a creel failure and holds the phase.
  If the withdraw had removed a hub-pause resume line, `State.ResumeLineWithdrawn` keeps it, so a later tick still clears the pause record's noticed mark.
- A pending `/compact` with no id has no outcome to read and is withdrawn once and dropped, as before.

The `ResumeLineWithdrawn` field comment in `state.go` and the compact-cycle paragraphs of `internal/orchengine/doc.go` describe the new behavior.
The creel fake in `fakecreel_test.go` now models the withdraw as creel does: queued injects become withdrawn and in-flight ones stay.

### Reed pane-listing separator

`internal/reedengine` joined the pane title, current command and start command in its `list-panes` formats with the ASCII unit separator.
tmux 3.4 escapes control characters in `-F` output, so the separator arrived as the literal text `\037`.
The tail never split: titles read as `name\037bash\037/usr/bin/bash`, title drift was reported on untouched panes, and a shell-only strand never read dead.
`paneTailSeparator` in `parse.go` is now a tab, which tmux passes through verbatim, and the formats in `overlay.go` and `ownwindow.go`, the package doc's format line and the test listings match it.

## Design decisions

- The wait logs at Debug, since it repeats every tick and is bounded by the handoff timeout.
- Creel, shuttle's send path and the reload sequence's step order are unchanged.
- Bound: if the delivery attempt the watcher waits on types the `/compact`, the session still compacts a second time.
  Only creel could prevent that, and the phase completes on it as on any typed `/compact`.
- The separator fix rides on this branch, as the parent's added decision records, because the reed tests it fixes fail on `main` and block this branch's verify.
  A tab inside a pane title would now split the tail, as a unit separator in one did before.

## Tests

`TestCompact_BoundaryWithdrawsAStillQueuedCompact` in `watcher_compact_test.go` covers:

- a boundary while the `/compact` is sending, which stays compacting with the step pending;
- the later outcomes injected, unconfirmed and back in the queue, and that only the last withdraws again;
- a withdrawn resume line with a compact still in flight;
- a failed re-read after the withdraw, with the next tick clearing the mark.

The existing reed parse tests and real-tmux naming and own-window tests cover the separator; `lyx gate test --tier 2 ./internal/reedengine ./internal/reedcli` passed against tmux 3.4.
`lyx gate test ./internal/orchengine` passed.

## To check by hand

- Against a live orch, request a compact while a creel delivery attempt is refused as busy, and confirm no second `/compact` reaches the pane and `lyx orch status` leaves `resuming`.
- On a machine with an older tmux, confirm strand pane titles read without a trailing command and a shell-only strand reads dead past its grace.

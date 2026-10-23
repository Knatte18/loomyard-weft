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
- An outcome still queued or sending keeps the step pending and the boundary unconsumed, and returns `errCompactInFlight`. `tickCompacting` treats it like a held withdraw, and the next tick withdraws again.
- Past the handoff timeout from the phase entry, an in-flight step is dropped with a warning and the phase completes, so the wait cannot last forever.
- A failed read after the withdraw is a creel failure and holds the phase.
  If the withdraw had removed a hub-pause resume line, `State.ResumeLineWithdrawn` keeps it, so a later tick still clears the pause record's noticed mark.
- A pending `/compact` with no id has no outcome to read and is withdrawn once and dropped, as before.

The `ResumeLineWithdrawn` field comment in `state.go` and the compact-cycle paragraphs of `internal/orchengine/doc.go` describe the new behavior.
The creel fake in `fakecreel_test.go` now models the withdraw as creel does: queued injects become withdrawn and in-flight ones stay.

## Design decisions

- The wait logs at Debug, since it repeats every tick and is bounded by the handoff timeout.
- Creel, shuttle's send path and the reload sequence's step order are unchanged.
- Bound: if the delivery attempt the watcher waits on types the `/compact`, the session still compacts a second time.
  Only creel could prevent that, and the phase completes on it as on any typed `/compact`.

## Tests

`TestCompact_BoundaryWithdrawsAStillQueuedCompact` in `watcher_compact_test.go` covers:

- a boundary while the `/compact` is sending, which stays compacting with the step pending;
- the later outcomes injected, unconfirmed and back in the queue, and that only the last withdraws again;
- a withdrawn resume line with a compact still in flight;
- a failed re-read after the withdraw, with the next tick clearing the mark.

`lyx gate test ./internal/orchengine` passed.
Tiers above that package were not run.

## To check by hand

- Against a live orch, request a compact while a creel delivery attempt is refused as busy, and confirm no second `/compact` reaches the pane and `lyx orch status` leaves `resuming`.
- `internal/orchengine/watcher.go` `withdrawQueuedCompact`: a failed `InjectOutcome` re-read after a successful withdraw must not lose a withdrawn resume line.
- `internal/orchengine/watcher_compact_test.go`: the re-read failure branch needs a test that fails without the `ResumeLineWithdrawn` save.
- `internal/orchengine/watcher_compact_test.go`: the `unconfirmed` outcome needs a row that ends the wait.
- `internal/orchengine/watcher_compact_test.go`: a withdrawn resume line combined with an in-flight compact must clear the mark and answer `errCompactInFlight`.

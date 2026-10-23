# Decision record — orch-compact-repeats

## Goal

Once a compaction the watcher asked for has landed, the watcher never lets a second `/compact` for the same request reach the orch session, and the reload it enters is not queued behind a stale `/compact`.

## Scope

- In: the compacting phase's handling of the watcher's own `/compact` inject once a qualifying compaction boundary is found (`internal/orchengine/watcher.go`), its tests, and the compact-cycle paragraph of `internal/orchengine/doc.go`.
- Out: creel's delivery worker and its withdraw verb, shuttle's send path, and the reload sequence's step order.

## Decisions

### The pending `/compact` must have left creel's queue before the boundary is consumed

Decision: after the compacting phase withdraws its still-queued `/compact`, it reads the pending inject's outcome again.
Only a withdrawn, injected or unconfirmed outcome lets the phase complete; one that still reads queued or sending keeps the step pending and the boundary unconsumed, and the next tick withdraws again.
Rationale: creel's withdraw removes only injects in its queue, and an inject a delivery attempt holds at that moment is left in place;
an attempt the session refuses as busy puts it back in the queue, and creel types it at the next idle, ahead of every later watcher inject.
The watcher dropped the pending step after any successful withdraw call, so such an inject survived as a second `/compact` typed into the resumed session, and the reload's color step waited behind it.
Bound: the hold lasts while one creel delivery attempt runs, which settles as typed or back in the queue;
an inject creel types in that attempt cannot be stopped from the watcher, and the phase completes on it as on any typed `/compact`.

### A pending `/compact` with no id keeps today's handling

Decision: a pending `/compact` whose id creel never answered, and which the journal lookup could not adopt, is withdrawn once and dropped, as now.
Rationale: with no id there is no outcome to read, and the tick's journal lookup has already run before the phase switch.

## Constraints

- `PATTERN-log-levels`: the wait for an in-flight inject repeats each tick, so it logs at Debug, never Warn.
- `PATTERN-test-economy`: the new behavior extends `TestCompact_BoundaryWithdrawsAStillQueuedCompact` rather than adding a test.
- The creel fake must model the withdraw as creel does: queued injects become withdrawn, in-flight ones are left.

## Auto-mode assumptions

- The reported run used a binary from before `e634efb9`, which had no boundary withdraw and requeued a pasted inject on a draft or foreign-text refusal; that path is already fixed, and this task closes the withdraw gap that remains.
- The reported `resuming` state with the color step pending is the downstream effect of the stale `/compact` ahead of it in creel's queue; the reload sequence itself needs no change.

## Open risks

- A delivery attempt that types the `/compact` while the watcher waits still compacts the session a second time; only creel could prevent that.

<!-- lyx:carry-over Darn-Review -->
- Darn-Review, round 1: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lyx/reviews/darn/round-1-review.md; fixer report: _lyx/reviews/darn/round-1-fixer-report.md.
  - no-test-post-withdraw-reread-failure (class scope, severity MEDIUM, open in rounds 1)
  - withdrawn-resume-line-lost-on-failed-reread (class design, severity MEDIUM, open in rounds 1)
<!-- /lyx:carry-over Darn-Review -->

## Acceptance criteria

- A boundary found while the pending `/compact` reads sending, with a withdraw that leaves it in flight, keeps the phase compacting with the step pending and the cycle count unchanged.
- On a later tick, the same inject read as injected completes the phase with no further withdraw, and read as back in the queue is withdrawn and then completes the phase.
- A boundary found while the pending `/compact` is queued still withdraws it and completes the phase in that tick.
- An injected `/compact` still makes no withdraw.

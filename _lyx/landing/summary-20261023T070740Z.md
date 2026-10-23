# Orch watcher waits out an in-flight /compact before reloading

When a compaction lands, the orch watcher's compacting phase withdraws its own `/compact` if creel still holds it, so the session never compacts twice for one request.
Creel's withdraw only removes injects in its queue.
It leaves an inject that a delivery attempt holds at that moment, and an attempt the session refuses as busy puts the inject back in the queue.
The watcher dropped its pending `/compact` after any successful withdraw call, so such an inject could survive the compaction.
Creel then typed it into the resumed session as a second `/compact`, ahead of the reload's color step, which kept the watcher in `resuming`.

## What changes

- **`withdrawQueuedCompact` (`internal/orchengine/watcher.go`)** reads the pending `/compact`'s outcome again after the withdraw.
  If it is withdrawn, injected or unconfirmed, the phase completes as before.
  If it still reads queued or sending, the step stays pending and the boundary stays unconsumed: the call answers the new `errCompactInFlight`, `tickCompacting` treats it like a held withdraw, and the next tick withdraws again.
  Each tick of that wait logs at Debug.
- **Timeout.**
  Past the handoff timeout from the phase entry, a `/compact` still in a delivery attempt is dropped with one Warn line and the phase completes.
  Without this, the compacting phase's own timeout would never be reached while a boundary qualifies.
- **The creel fake (`fakecreel_test.go`)** now withdraws the way creel does.
  Injects that read queued become withdrawn, and ones the fake reads as sending are left alone.
- **Tests.**
  `TestCompact_BoundaryWithdrawsAStillQueuedCompact` covers three ends of an inject caught mid-delivery: it is typed, it goes back to the queue and is withdrawn, or it stays sending past the timeout.
  Each case checks that the reload, not another `/compact`, is queued last.
- **`internal/orchengine/doc.go`**: the compact-cycle section describes the re-read, the retry and the timeout.

## Design decisions to review

- The fix lives in the watcher, not in creel's withdraw.
  The attempt that holds the inject may already have typed it, so creel cannot safely drop it mid-send.
  Re-reading the outcome gives the watcher the same safety without changing creel's verb.
- A pending `/compact` whose id creel never answered is still withdrawn once and dropped, because it has no outcome to read.
- Bound: an inject that the held attempt types still compacts the session a second time.
  The watcher cannot stop a keystroke creel is already sending.
- The reported incident ran a binary from before `e634efb9`.
  That binary had no boundary withdraw, and it requeued a pasted inject that creel refused for a draft or foreign text in the box.
  Both of those are fixed already.
  This change closes the remaining path to the same symptom.

```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals the fixer carried out (F1–F5), as fix-proposal lens.

Findings

1. **LOW**: `inbound.go`, `handle` and `awayPending` (F3 proposal).
   - Summary: the pending set can mark an orch that the turning message never woke. This happens when the turn is made by one message and the post by another.
   - Evidence:
     - A message turns the presence and its post fails, so `awayPending[orch]` is set.
     - The retry window runs out. `retryOrGiveUp` returns `(true, nil)`, `posted` is false, and the `err == nil` case clears the entry. This path is handled.
     - A failure path that returns an error forever is not cleared. Examples are a presence write that succeeds on the first attempt followed by an `advance` error, or a drain that stops on a rate limit before the retry.
     - In those cases the entry stays pending. A later, different message to the same orch that posts, once the presence is already slack, gets `awayFrom[orch]=true`.
     - That later message did not turn the presence, but its post does wake the orch. The skip is still right for that post.
     - The harmful case is a presence that went back to terminal in between. Then the next message turns it again, so `turned` is true anyway and nothing is lost.
   - Net: no wrong behaviour found beyond a harmless stale entry. This is noted only because no test pins the stale case.

2. **LOW**: `relay_test.go`, the "OrchKeptByAWakeThatSentNoNotice…" row (F1c).
   - Summary: the row waits on `beamReads >= reads+2`, which may not prove that the forwarding wake finished.
   - Evidence:
     - `beamReads` is also incremented by other callers of `hub.Beams`, for example outbound or keeper paths. The relay's `Hub.Beams` wrapper counts every call.
     - Two reads could therefore occur inside one wake, or from the outbound wake. Then the wake that forwards the message may still be running when `setPresence(slack)` is written.
     - If that wake then reads the presence as slack, it takes `awayFrom={prime}` and skips prime. The expected notice list would fail, so the row can flake.
     - I did not check how many times a single wake calls `Beams`. `wake` calls it once, and `ensureBeamChannels` or `retire` may call it again.

3. **LOW**: `outbound_test.go`, the "EmptyPollOfExactlyTheRaisedFloor…" row (F5).
   - Summary: the row relies on the `h.mustRead()` after the rate limit returning `2*Floor` through an activity pass. It does not assert that.
   - Evidence: the second step posts the batch successfully, and the `h.mustRead()` result is discarded. If the floor were not raised, the "exactly the floor" assertion `2*Floor` would no longer hold. The row would then fail rather than pass silently, so this is a weak-pin note only.

The F4 doc rewording and the F2 `posted` flag hold up against the cases I built.

Calls made: none. My context already held every changed file.

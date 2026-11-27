```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the correctness of the code this round's fix changed (lens fix-code-correctness).

1. **MEDIUM** — `internal/runpusher/parent.go`, the new `HandOvers` persistence with `payOwedSteps`: an owed hand-over can be left with nothing that retries it.
   - Both `settleOnOrigin` and `dropLandedElsewhere` now remove the landing and write `HandOvers[run]`, and the hand-over itself happens later, in `payOwedSteps`.
   - Take the only landing on the landing target, owned elsewhere. Any failure before `HandOver` runs leaves `HandOvers` with no landing and no held-back record. Examples: `RunOwner` fails in `payOwedSteps`, `WorkUnderOtherCommit` or `orderedLandings` fails in `dropLandedElsewhere`, `clearUnlessDropped` fails, or the process dies.
   - `settleOnOrigin` has also just cleared any record through `clearUnlessDropped`.
   - Nothing then reaches it. `targetAhead` reads only `Landings` and `UnsettledLandings`, so `pushUntilLevel` goes level and exits even after the `owed(err)` return, and `StartIfAhead` starts nothing. `restartHubPushers` runs `SettleRefused` only while the target reads refused.
   - The hand-over then waits for an unrelated later landing. F2's fix asks for a retry on every pass and in `SettleRefused`.
   - The fix would be to have `targetAhead` or the tick count `HandOvers` as work.

2. **LOW** — `internal/webstercli/reset.go`, `pushPairRecords`: a busy push lock leaves no owed record.
   - `fabricSync` has just started the quiet pusher, which pushes with `OwedUnrecorded` and holds the pair push lock.
   - The synchronous `PushRun` waits `StatusPushLockWait`. If the lock is still held, it returns `ErrPushLockBusy` ("neither side was pushed", `pushanchored.go` `pushLockBusyError`), which writes no failed-push record.
   - The verb then reports `push_error`, but no record holds the run. This contradicts the new help text ("recorded as a failed push, which holds the run"), the `pushPairRecords` doc and the new `PATTERN-run-fsm` line.
   - This happens only when the pusher's push outlasts the wait, for example on a hanging network.

I found no other defect in the changed functions I traced, and no added defect test that would pass with its defect still in place.

Tool calls:
- `grep ErrPushLockBusy internal/fabricengine/pushanchored.go`: my context did not show whether a busy push lock writes a failed-push record.

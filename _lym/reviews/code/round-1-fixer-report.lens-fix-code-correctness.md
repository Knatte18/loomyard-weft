```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the fix's code correctness (lens fix-code-correctness).

**Findings**

1. **MEDIUM — `internal/creelengine/sweep.go`, the new `isOpenQuestion(msg) && isStrandName(msg.From)` clause in `sweepSubjects`, and the new asker loop in `releaseGoneHolder`.**
   - The sweep treats any strand name absent from this hub's strand listing as gone.
   - A question that arrived from another hub or machine carries a `From` that is a strand name of that other hub, which is never in the local listing. `releaseGoneHolder` already handles this case: `sweep.go:239` tests `msg.SendingHub != ""`.
   - Such a question is held by a local strand, so `isOpenQuestion` is true. Its asker is now a sweep subject, counts as absent, and after `goneAfterSweeps` (2) sweeps `releaseGoneHolder(<remote asker>)` settles it unanswered with `asker-gone`.
   - That ends the chase of a legitimately open cross-hub question after roughly 40 s. Before the fix, only holders were ever swept.
   - The asker clause and loop need a guard for `msg.SendingHub == ""` and `msg.SendingMachine == ""`. No added test covers a cross-hub question.

2. **LOW — `internal/shuttleengine/replyhold_test.go`, `TestRun_Wait_ReplyHold_ReleaseOfAFailingGateRepromptsOnce`.**
   - The test is meant to pin `run.replyHold = nil` in `releaseReplyHold`. It probably passes with that line removed.
   - After a delivered re-prompt there is nothing pending or unsent, so the `replyHold != nil` branch shadows no re-evaluation.
   - A stale hold would re-run `handleGatedBoundary` only if `gateAtBoundary` is still true. The code I saw sets it false only on the pending-send path (`wait.go:603`). I did not see the re-prompt-success tail.
   - If `gateAtBoundary` is false after a delivered re-prompt, the F5 mutation survivor remains.
   - The test would need a state where the stale hold changes an observable result. One option is a re-prompt that fails to send (`unsentReprompt`) while the hold is stale.

3. **LOW — `internal/shuttleengine/replyhold.go`, `agentBusy`.**
   - Release now also requires `Startup == StartupReady && IdleSession`.
   - On a real Claude pane, an operator draft in the input box, or a dialog or picker, reads as not idle.
   - A question that settled with no reply then holds the run until the run deadline instead of finalizing. Before the fix, two quiet probes were enough.
   - Bounded by the deadline, but it is a behaviour change the summary does not mention.

Checked and sound:
- `AwaitsReplyProbe` fallback: only `ErrStrandClaimOutsidePane` falls through, and other errors propagate.
- `agentBusy` with an engine that is not a `SessionCycler`, and with a capture error: both read as not busy.
- The new `busyUntil` and turn-start rows would fail without the `agentBusy` check and without the turn-start hold clear.
- The F6 rows fail without the `EndedUnreadBy` clause and without the `original.From == strand` check.
- `HasReplyProbe` and the burler `runner` field are consistent with their callers.

Tool calls made: none; the context already held everything this lens needed.

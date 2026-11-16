```yaml
lens: guard-reach
outcome: reported
```

Task: guard-reach lens. For each guard, refusal, branch or test row the cards add, can a real entry point reach it?

**Per guard: reachable or not**

1. **Card 5, `takeoverStop` in `retryWeftPush`:** reachable. A rejected records push calls `settleRejection`, which runs `retry` at once. The stop sits after the fetch and before `moveRecordStop` and `upstreamAnchorStop`.
2. **Card 5, "takeover is returned unrecorded" (`pushClassified`):** not reachable as written. The non-retryable error from `settleRejection` goes to `classifyFailure`, matches no case, and is recorded as `other`.
3. **Card 5, `ErrHandedOver` when a mark names the machine:** reachable once card 6 writes the mark.
4. **Card 5, the `CoalescePushRequest` anchor-change row:** reachable. The row passes even with the recording defect, because nothing lands.
5. **Card 3, the `teardown_lock` owed push:** reachable with a positive `TeardownWait` against an exclusive holder. Card 8's owed branch only sees it if the returned error is a `*PushFailedError`.
6. **Card 3, records side deferred or ended, so no code side:** the branch inside `PushPairAnchored` is reachable. Its "not landed" state is not passed up to card 8's seam.
7. **Card 7, owner's tip guard (`resetUnnamedTip`):** reachable through a rejected code push, then `repushOverOriginContained`, then `HasUnpulled`. I did not confirm whether `holdBackRewound` holds the push back first when the given-up tip was already fetched; `reflog.go` past line 113 is unread.
8. **Card 6, `pushOwedCodeTip` push and re-read reset:** reachable inside `HandOverTips`; the `owedTipPushedHook` row drives the reset.
9. **Card 11, owner read after the merge:** reachable on a re-step. `ownerChecks` skips when the parent holds the work, but `pushParent` re-asks `OwnedElsewhere`.
10. **Card 11, `ArchiveParentMerge` after a rejected push:** reachable only in a narrow state: the Bolt reads this machine as owner while origin's parent already holds the work. One case is re-adoption after the adopter landed. `syncParent` and `ownerChecks` both skip once `ParentHoldsTaskWork` is true.
11. **Card 12, give-up past the grace:** reachable. It also fires when this machine holds the tip itself (see finding 5).
12. **Card 12, single-transition "returns false" branch:** reachable.
13. **Card 8, owed retry on transport or `ErrPushLockBusy`:** reachable.
14. **Card 8, "bare `ErrPairPushRunning` is never seen":** holds for both transition seams. Card 12's `PushRun` passes no `TeardownWait`, but it only logs, so this is not a defect.
15. **Card 9, `owedIntentPush`:** reachable at tier0.
16. **Card 9, the new end-only intent write in `endRunPhase`:** the state it changes does not occur. Today's code already writes and commits the intent before `EndForCancel` on the end-only path.
17. **Card 1, the `push_owed` and `code_tip` refusals:** reachable through `TestAct`, and in production once card 7 fills the inputs. The `code_tip` way forward can be unreachable (see finding 6).
18. **Card 13, the `Owes` cell:** reachable.

**Findings**

1. **BLOCKING, consistency.** Location: card 5, `pushClassified` (not among its targets).
   - Summary: the takeover is never "returned unrecorded" as card 5 claims.
   - Evidence: `internal/fabricengine/pushclassified.go:133-145`. After `settleRejection` returns `retryErr`, `classifyFailure` matches no case, and line 145 calls `job.fail(PushFailureOther, …)`. That writes a failed-push record and a creel notice and wraps the error in a `*PushFailedError`.
   - Effect: card 5's row "no failed-push record stands" cannot pass. In `coalescePushSides`, `pushSideDetached` (`coalesce.go:64-66`) swallows the `*PushFailedError` and returns nil, so the detached child never sees the takeover.
   - Fix: card 5 adds `internal/fabricengine#pushClassified` (or `pushJob.classifyFailure`) and passes `*TakeoverError` and `ErrHandedOver` through unrecorded.

2. **BLOCKING, decision (D1).** Location: card 3, `PushPairAnchored`, and card 8, `statuscommit.New`.
   - Summary: a records side that is deferred or ended returns nil, so the seam treats the push as landed and starts the next producer.
   - Evidence: `pushclassified.go:134-136` returns nil from `job.deferral()`, and `pushclassified.go:423-424` returns nil for an ended branch. Card 3's own row says "a records side deferred by a held weft write lock: the call returns nil".
   - Card 8 maps nil to "landed: release, return nil, and the next producer starts".
   - The `deferred` record class is not owed, so card 7's `ReadRun` does not hold the run either.
   - D1 says "the next producer starts only once every side has landed". The records-landed check card 3 adds is enforced for the code side only, not for the transition.

3. **MEDIUM, consistency.** Location: card 3 (`acquireTeardownShared` wait-out) and card 8 (`IsPushOwed`).
   - Summary: card 3 says the push "returns the error wrapping `ErrPairPushRunning`". Card 8's `IsPushOwed` counts only a `*PushFailedError` of class `teardown_lock` and treats a bare `ErrPairPushRunning` as not owed.
   - Neither card says the wait-out error is a `*PushFailedError`, and card 3's test row asserts only `ErrPairPushRunning`.
   - Effect: an implementation that follows card 3's words makes card 8 halt the run instead of owing the push. That breaks D1's criterion "a transition's push that waits on a held teardown lock past the push lock wait turns owed".
   - Evidence: `teardownlock.go:119`, where today's refusal is a plain `fmt.Errorf` wrapping `ErrPairPushRunning`.

4. **MEDIUM, consistency.** Location: card 9, `endRunPhase`.
   - Summary: the card's premise is false. It says "today `endRunPhase` then stops with nothing written", but today's code already runs `WriteIntent` and `CommitRecord` before `EndForCancel` whatever `endOnly` is.
   - Effect: the "now writes the cancel intent … on the end-only path too" change targets a state that does not occur. `TestCancelRun`'s "now expect `WriteIntent`, `CommitRecord`" rows may already hold, so they would never be seen failing.
   - Evidence: `internal/battencli/cancel.go:313-316` (`judgeOrigin` only sets `endOnly`), `:370-379` (intent written and committed), `:412-417` (end-only return after them).

5. **MEDIUM, design.** Location: card 12, `settleCodeTip` give-up branch.
   - Summary: the give-up past the grace is reachable when the owing machine is this machine and the tip sits in its own task worktree.
   - How: the owner's own code side fails with a class that is not owed (`other` or `not_fast_forward`), so `PushOwed` stays false and the `code_tip` verdict follows. After 30 minutes, `ReturnToPreviousTransition` discards the transition. No branch first pushes a tip this repository holds, as card 6's `pushOwedCodeTip` does for the hand-over.
   - Evidence: card 12's Intent branches, and `runfsm/act.go:84-90` for the act order.

6. **MEDIUM, design.** Location: card 7, `ReadRun` filling `CodeTipMissing`, and card 1's `code_tip` refusal.
   - Summary: cancel and resume now meet the `code_tip` wait, which their `Aside` does not drop; card 7's ImpactSummary says so.
   - When origin's records hold a single transition, card 12 never gives up. The run then waits for good and `lym batten cancel &lt;slug&gt;` is refused indefinitely.
   - The refusal's way forward, "run again once the code push lands", cannot be reached if the owing machine is gone.
   - Evidence: `internal/battencli/cancel.go:332-338`, where the verdict refusal comes before any cancel work.

**Tool calls**
- Grep of `classifyRewrite` in `internal/fabricengine/reflog.go`: to check whether `holdBackRewound` pre-empts card 7's owner guard; the manifest lacked `reflog.go`.
- Read of `internal/battencli/cancel.go:288-427` (`endRunPhase`): to trace card 9's end-only path; the manifest held only that file's declarations.

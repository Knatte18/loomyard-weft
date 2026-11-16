```yaml
lens: guard-reach
outcome: reported
```

Lens guard-reach: tracing whether each guard, refusal, branch and test row the plan adds can be reached from a real entry point.

**Findings**

1. **BLOCKING**: Cards 3 and 4, D1 records-first order. A deferred records side still sends the code side.
   - `pushClassified` returns nil for a deferred records push (`return job.deferral()`, `internal/fabricengine/pushclassified.go:136`). The same holds for an ended branch (`checkOrigin` returns proceed false with a nil error, `:423-424`).
   - Card 3's `pushSide` and card 4's `pushSideDetached` treat that nil as landed. The code side then goes out while the records side has not landed.
   - A busy weft write lock (`retryWeftPush`, `pushclassified.go:614-615`) or a dirty replay (`ErrReplayDirty`, `:667-668`) leads to that deferral. The "withhold code side" guard misses this path.
   - Fix: pushClassified must tell the caller apart a deferral from a landing.

2. **BLOCKING**: Card 7, the owner's guard (D2, "also when the owing machine dies before its re-read"). The guard can never fire in the case it exists for.
   - `resetUnnamedTip` resets only a tip that "no `Warp-SHA` on the fetched records branch's history equals".
   - `ReturnToPreviousTransition` (card 12) adds a new commit and leaves the given-up transition's commit in history. That commit's `Warp-SHA` still names the given-up tip, so the tip always reads as named.
   - The test should be against the last transition, as card 6's `pushOwedCodeTip` re-read does.

3. **BLOCKING**: Card 6 step 3 and card 11, D2 criterion "old owner merged into its local parent … level with its upstream, its merge under an archive tag".
   - `HandOverTips` archives the parent only when `HandOverRequest.Parent` is set.
   - No production path sets it. `PushRun` calls `HandOver(prime, slug, entry, push)`, and that signature carries no parent. Card 11's `Deps.HandOver` passes only owner and subject.
   - So `archiveParentMerge` runs only in `TestHandOverTips`. In card 11's hand-over before the parent push, the local merge is neither archived nor reset.

4. **MEDIUM**: Card 11, the hand-over before the merge. It is preempted by the existing owner check.
   - `Finalize.ownerChecks` (`finalize_owner.go:23`, step 4 of `landUnderLock`) already halts Stuck with `ownerHalt` when the run is owned elsewhere. It runs before the merge, and the card does not touch it.
   - The new `handOverIfOwnedElsewhere` before the merge is reached only by fakes that disagree with `Deps.Reconcile`. In production the run halts instead of handing over.
   - The planned `TestFinalize_OwnerChecks` row ("hands over and merges nothing") can pass only with inconsistent fakes.

5. **MEDIUM**: Cards 6 and 11, the hand-over mark skip. It is unreachable after a hand-over that the Finalize starts.
   - Card 11 builds a takeover entry from the Bolt owner, so the mark records a made-up `At`.
   - Card 5 skips a hand-over only for a mark that names the same machine and `At` as origin's real entry. A later rejected push therefore hands over again: it ends processes, archives and writes a new mark.

6. **MEDIUM**: Card 9, cancel with origin unreachable (D1 criterion). The "owed, re-run" branch is effectively unreachable.
   - `endRunPhase` calls `judgeOrigin` before `WriteIntent` (`internal/battencli/cancel.go:313` before `:371`).
   - An unreachable origin sets `endOnly` (`cancel.go:604` and `:609`), and the cancel stops end-only without writing the intent. The card leaves that path as it is.
   - So with origin unreachable the verb never "acts here at once and owes its push". `clearRunPhase`'s owed branch is reached only if origin drops between the judgement and the push.
   - Card 8 also routes `commitCancelRecord` through a synchronous `Push` without saying what an owed push does before `EndForCancel`.

7. **MEDIUM**: Card 9 test row, "hub's origin pointed at a missing path". It cannot reach the owed branch.
   - A missing-path remote fails with "does not appear to be a git repository". That matches no `transportPhrases` entry (`internal/gitexec/transport.go:22-31`), so the push is classed `other`, not `transport`.
   - `IsPushOwed` answers false and the pause verb fails, which is the opposite of what the row asserts. Use an unreachable host or port instead.

8. **MEDIUM**: Card 12 with card 7, the code-tip give-up. Stale fetched refs can trigger it.
   - `OriginCodeTip` reads only fetched refs, "with no network call".
   - Neither `settleCodeTip` nor `DobbyRuns` fetches the code branch. Card 12's `ReturnToPreviousTransition` fetches only the records branch, and `dobbyreads.go` has no fetch.
   - On the owner, a tip that another machine's catch-up already pushed keeps reading missing until something fetches the code branch. After 30 minutes the owner gives up a tip origin holds. The guarded state is reached falsely.

9. **MEDIUM**: Card 7, `FailedPushesOwedBy`. The owed-push wait reaches every run.
   - It counts every `transport` or `teardown_lock` record on the prime's records branch as owed by every slug.
   - So one owed push on the prime's records branch makes every run's verdict `push_owed` under the dobby's `Aside{}` (`battencli/dobbyreads.go:191`). This includes runs whose records are all on origin, and a failure of a detached push that is not any run's (the prime's push in `tendercli/deps.go:144`).
   - Card 7 has no row that pins this reach.

10. **MEDIUM**: Card 10 with card 11, a landing's `done` pushed synchronously (D1). The new error is swallowed.
    - `SetStatusPushed` returns `ErrOwnerOriginUnreachable`, but `Finalize.markTaskDone` only logs a failure, per the exploration.
    - Card 11 does not make it fail the step, so the writer goes on with the `done` unpushed, against "pushes synchronously before its writer goes on".

11. **LOW**: Card 3 with card 8, batten's `TeardownWait`.
    - Batten's transition push is the prime's records push, which takes no teardown lock (`pushanchored.go:110`, `TeardownSlug` empty for the prime).
    - So the `TeardownWait: StatusPushLockWait` card 8 passes in `battenCommitStatusDeps` is never reached. Only loom's pair pushes reach the `teardown_lock` owed branch.

12. **LOW**: Card 5, `CoalescePushBothAt` with zero `MachineNames`. Its only production-file caller is `internal/hubforge/hub.go:424`, a test-support package, so the unguarded site does not recur in production. This entry only confirms the card's claim, which the lens report needs as evidence; it is no defect.

**Tool calls**
- grep for `CoalescePushBothAt` callers, `cancel.go` symbol lines, the start of `gitexec.IsTransportFailure`, and fetches in the dobby read and loop. The context lacked the callers, the order of the cancel judgement and the transport classification.
- Read of `transportPhrases` and `cancel.go` 300-330 and 591-620. The context lacked the phrase list and where `judgeOrigin` sits relative to `WriteIntent`.

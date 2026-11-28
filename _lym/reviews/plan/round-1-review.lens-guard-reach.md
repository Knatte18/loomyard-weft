```yaml
lens: guard-reach
outcome: reported
```

guard-reach: whether each guard, refusal, branch and test row the cards add can be reached, and whether each guarded site recurs on a path no card guards.

**Per guard: reachable or not**
- Card 1 `RefuseUnsupplied` and `TestRefuseUnsupplied`: reachable. Card 4's `refuseUnsuppliedSeams` and card 9's `OriginJudge` call it when a setter leaves a seam unset.
- Card 3 `snapshotReader` `FieldSeams` case: reachable, never fails. The `TestSnapshotReader_ReadsAHub` row is reachable because the sequential setter tests restore before the parallel ones run.
- Card 4 refusals in `OriginRunStatus`, `OriginCodeTip` and `originStatusBytesAtTip`: reachable through `TestInstallRequiredSeams`' setters. The guarded read recurs, unguarded before acting, on the push and handover paths (F2).
- Card 4 `MarkBeamVerdict` and `ReadOriginForVerdict`: reachable.
- Card 4 `RemoteOnly` no-origin branch: reachable only if the engine skips the tips read (F5).
- Card 4 `TestInstallRequiredSeams` run-lock check: it reaches the busy branch, but not through the adapter until card 11 (F3).
- Card 5 text rules, `MergeFailedPush` notify branch, filters: reachable.
- Card 6 `DescribePushLockHolder` clauses and `FlaggedLoopNext` branches: reachable.
- Card 7 transport loop: reachable, but it also catches `ErrPushRetriesExhausted` and adds rounds (F1).
- Card 7 `OutcomeProtected`: reachable on the first push. It shadows today's rejection retry and judgment fetch (F4).
- Card 7 `OutcomeTakeover`, `OutcomeDeferred` (`errRetryDeferred`), `OutcomeOriginAhead`, the ended-entry `ActionRefDelete`, the namespace failure and `UnseededBoardRefusal`: reachable. The ended-entry row is reachable only as a core row, since fabric tests record no run.
- Card 8 `RecordsSideLanding` and `RecordsLandingVerdict` branches, `BranchAhead`, `BranchHeldOnOrigin`, `RunAhead`, `TipsHeldOnOrigin`: reachable.
- Card 9 `OriginJudge` pre-lock refusals: reachable. The `SeamRunLocks` refusal fires for every moving hub scope, including one whose prime moves nothing and so takes no prime lock (F6).
- Card 9 transport retry, `Unreachable` and the moved-heads fetch: reachable. `TestReconcileOrigin_RefusesAnUnsuppliedSeam`: reachable.
- Card 10 `EndedTeardowns` seam call, failing teardown, judge-only, own run, no local part, no seam: reachable.
- Card 11 `MoveVerdict` precedence, live probe at execution, `TestReconcileOrigin_PrimeLockHeld`: reachable.
- Card 12 `TestBoltReplayers` no-replayer row: reachable. Its own `writeStatus` record makes `recordedPairs` find the pair, so `readOriginOwners` reaches the ledger. Kept moves and judge rows that record no run reach no ledger read, as intended.

**Findings**

F1. BLOCKING, design, `_lym/plan/07-push-side-core.md` (outcomeClassOf bullet and the transport rule)
- Summary: card 7 maps `gitrepo.ErrPushRetriesExhausted` to `OutcomeTransportFailure`, and `PushSide` pushes a transport failure again after each backoff delay.
- Today `retryTransportPush` re-pushes only when `gitexec.IsTransportFailure` holds.
- `PushRebaseFree` returns an exhausted inner retry wrapping `ErrPushRetriesExhausted`, not as a transport `GitError`.
- So that error now reaches a loop that today never re-pushes it, each round with its own bounded inner retry. That breaks D3's "no loop runs more rounds than today's code retries" and D2's git-process bound.
- The card's note covers only read rows.
- Evidence: `internal/fabricengine/pushclassified.go:205-221` (`retryTransportPush` tests `IsTransportFailure`), `:647-648` (`classifyFailure` records it as transport without a re-push), `internal/gitrepo/push.go:253-256`, `internal/gitexec/transport.go:114-121`.

F2. MEDIUM, decision, `_lym/plan/04-origin-reads-seams.md` (`refuseUnsuppliedSeams` and `originStatusAtTip` bullets)
- Summary: the card calls `refuseUnsuppliedSeams` "the pre-lock refusal every verb of this part runs before it reads or acts". Cards 7 and 8 never run it.
- Card 4 routes `originStatusAtTip` through `originStatusBytesAtTip`, which refuses on an unset run-records seam.
- `ReadUpstreamTakeover` (`takeover.go:95`) reads it inside `retryWeftPush`'s `takeoverStop`, after the first push has run.
- An unset seam then surfaces mid-flow as a retry failure. It is joined to the rejection and written as an `other` failed-push record with a mail, not refused before acting as D13 requires.
- The same site recurs unguarded in `handover.go:200,215` (`OriginParentRunStatus`, `OriginBattenStatus`).
- Evidence: `internal/fabricengine/pushclassified.go:984-991` and `:341-344`, `internal/fabricengine/takeover.go:95`, `internal/fabricengine/handover.go:200,215`.

F3. MEDIUM, design, `_lym/plan/04-origin-reads-seams.md` (`TestInstallRequiredSeams`, the run-lock check)
- Summary: the check holds the lock at `shedrun.PrimeRunLock` and asserts that the reconcile reads the prime's branches as busy.
- In card 4's state, `tryPrimeLock` still calls `shedrun.PrimeRunLock` directly; card 11 moves it onto `RunLocks`. So the row passes without reaching the run-lock adapter it claims to check.
- Evidence: `internal/fabricengine/originreconcilemoves.go:44-45`; card 11's `tryPrimeLock` bullet.

F4. BLOCKING, decision, `_lym/plan/07-push-side-core.md` (`OutcomeProtected` ahead of rejected, and "protected is `other`")
- Summary: a protected-branch refusal wraps `ErrPushRejected`. Today the first protected push runs `settleRejection`'s retry once, which is `repushOverOriginContained`'s fetch and push.
- `failRejection` then runs its judgment fetch and records "…; a fetch found origin holding nothing this branch lacks", or `not_fast_forward` when origin is ahead.
- With protected mapped ahead of rejected and stopping at once, the first push's outcome is `OutcomeProtected`. The rejection-retry branch and the judgment read become unreachable for it.
- The record's error text, its class when origin is ahead, and the git process count all change, against D5.
- Evidence: `internal/fabricengine/pushclassified.go:164-176`, `:361`, `:639-640`, `:664-690`; `internal/gitrepo/push.go:292`; `internal/fabricengine/pushclasses_integration_test.go:466-483`. That row asserts only the class and the zero sleep count, so it does not catch the change.

F5. LOW, design, `_lym/plan/04-origin-reads-seams.md` (`RemoteOnly(hasOrigin, tips, branch)`)
- Summary: one pure call over `hasOrigin` and `tips` needs the tips read before the rule answers.
- Today the no-origin early return comes before `CurrentBranch`, `CurrentSHA` and `remoteBranchTips`.
- An engine that reads the tips first meets a read error, or an extra network call, before the no-origin branch can answer. The card names no phase split.
- Evidence: `internal/fabricengine/remoteonly.go:14-34`.

F6. LOW, scope, `_lym/plan/09-origin-judge-core.md` (phase 1)
- Summary: `OriginJudge` refuses an unsupplied `SeamRunLocks` for every moving hub scope.
- `movePrime` takes the prime lock only when a prime branch needs a move, so a reconcile whose run would never use the seam is refused. D13 refuses only a verb "whose run would use one".
- Evidence: `internal/fabricengine/originreconcilemoves.go:358-389`.

Outside my lens, for the handler: card 10 does not say which `runRule`'s under-lock phase it uses. If it is `OriginJudge`'s, the remote-refs locks would stay held through the teardowns, while today `judgeAgainstOrigin` releases them first (`originreconcile.go:615-622`).

Tool calls:
1. `internal/gitrepo/push.go` (`PushBranch` and `PushRebaseFree` docs), the `ErrPushRetriesExhausted` sites and `gitexec.IsTransportFailure`: needed to see whether an exhausted push counts as transport today.
2. `boltreplayers_integration_test.go`, the no-replayer row: needed to trace whether card 12's rewrite still reaches both ledger reads.

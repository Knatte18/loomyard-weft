# Card 2 — fabric-push-retry

**Edit:**
- `internal/fabricengine#pushSeam.pushBranchWithRetry`
- `internal/fabricengine#archiveWeftTip`
- `internal/fabricengine#pushRebaseFreeLogged`
- `internal/fabricengine#TestPushBranchWithRetry_RetriesOnlyTransientRefusalsUpToTheBound`
- `internal/fabricengine/doc.go`
- `internal/landingshed#TestPublish_StuckBeforePullRequest`

**Delete:**
- `internal/fabricengine#transientPushRefusal`
- `internal/fabricengine#pushMaxAttempts`
- `internal/fabricengine#pushBaseBackoff`
- `internal/fabricengine#pushSeam.sleepFor`
- `internal/fabricengine#TestTransientPushRefusal_ClassifiesByTheReasonOnThePushedRefsLine`

**Create:**
- `plan:internal/fabricengine#unpushedOutcome` -> `func unpushedOutcome(path string, err error) error`
- `plan:internal/fabricengine#TestUnpushedOutcome_WarnsAndLeavesCommitsForARejectedOrExhaustedPush` -> `func TestUnpushedOutcome_WarnsAndLeavesCommitsForARejectedOrExhaustedPush(t *testing.T)`

**Uses:**
- `plan:internal/gitrepo#PushWithRetry`
- `plan:internal/gitrepo#ErrPushRetriesExhausted`

**Intent:**
Fabric's own pushes that do not go through `gitrepo`'s primitives take card 1's retry, and fabric's private copy of it goes away (D6).

`pushSeam.pushBranchWithRetry` keeps its `git push -u origin <branch>` through `pushSeam.runPush`, and hands the loop to `gitrepo.PushWithRetry` with `branch` as the ref and the seam's `sleep` field (nil is gitrepo's production sleep).
`transientPushRefusal`, `pushMaxAttempts`, `pushBaseBackoff` and `pushSeam.sleepFor` are deleted, and `internal/fabricengine/pushretry.go`'s header says Add's branch pushes delegate to gitrepo's retry.
`archiveWeftTip`'s tag push runs through `gitrepo.PushWithRetry` with the tag ref and a nil sleep, so the archive push retries the same two refusal forms.

`pushRebaseFreeLogged` delegates the outcome of its `PushRebaseFree` call to the new `unpushedOutcome`: nil stays nil;
`gitrepo.ErrPushRejected` logs today's divergence Warn and returns nil;
an error wrapping `gitrepo.ErrPushRetriesExhausted` logs a Warn that the remote kept refusing the push and the commits are left unpushed for a later push, and returns nil;
any other error is returned.
So a coalesced push, the `fabric sync` push child and the verbs on them keep today's outcome for this refusal, which used to reach the divergence branch through the `rejected` substring, and gain no new error.

`internal/fabricengine/doc.go`'s Add push-retry paragraph is rewritten: every plain fabric push retries a remote `(failed)` or `cannot lock ref` refusal through gitrepo, Add's branch pushes and the archive tag push included, and the coalesced push Warns and leaves the commits unpushed once the attempts run out.

Tests:
`TestPushBranchWithRetry_RetriesOnlyTransientRefusalsUpToTheBound` gains a remote `cannot lock ref` row, and its persistent-refusal row asserts the returned error satisfies `errors.Is` for both the last push error and `gitrepo.ErrPushRetriesExhausted` instead of identity with the last error;
its classifier companion, `TestTransientPushRefusal_ClassifiesByTheReasonOnThePushedRefsLine`, is deleted with the function, since card 1's classifier test covers those rows;
the new `TestUnpushedOutcome_WarnsAndLeavesCommitsForARejectedOrExhaustedPush` table covers the four outcomes above;
`TestPublish_StuckBeforePullRequest` gains a row whose push error wraps `gitrepo.ErrPushRetriesExhausted`, asserting Publish is Stuck with `push failed: ...` and not the rejected-push reason, which needs no Publish change since only `ErrPushRejected` takes that branch.
The test count of `internal/fabricengine` is unchanged: one test deleted, one created.

**ImpactSummary:** fabricengine's Add, archive and coalesced pushes switch to gitrepo's retry; every deleted symbol's only callers are in `pushretry.go` and its test.

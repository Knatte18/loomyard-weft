# Card 1 — gitrepo-push-retry

**Create:**
- `internal/gitrepo#ErrPushRetriesExhausted`
- `internal/gitrepo#retryablePushRefusal`
- `internal/gitrepo#PushWithRetry`
- `internal/gitrepo#pushMaxAttempts`
- `internal/gitrepo#pushBaseBackoff`
- `internal/gitrepo/pushretry_test.go`
- `internal/gitrepo/pushretry_integration_test.go`

**Edit:**
- `internal/gitrepo#Repo`
- `internal/gitrepo#Repo.pushWithRebaseRetry`
- `internal/gitrepo#Repo.PushRebaseFree`
- `internal/gitrepo/doc.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:**
Decision D6 moves fabric's bounded push retry into the plain, never-forced push primitives, and this card builds it in `internal/gitrepo`, in a new file `internal/gitrepo/pushretry.go`.

`retryablePushRefusal`, unexported since no caller outside `internal/gitrepo` needs it, reports whether `err` is a `*gitexec.GitError` whose stderr holds a `! [remote rejected] <src> -> <dst> (<reason>)` line for `ref`, and returns that line.
The line belongs to `ref` when both `<src>` and `<dst>` equal `ref` in its short form or its full `refs/heads/` or `refs/tags/` form; every other line is ignored.
It is retryable when the reason is exactly `(failed)` or contains `cannot lock ref`; the reason may hold spaces, so the line is parsed up to the arrow and then the parenthesised rest, not split into a fixed field count.
A local `error: cannot lock ref ...` with no remote-rejected line is not retryable, and neither is a hook refusal, a `! [rejected]` line or any non-`GitError`.
An empty `ref` matches no line.

`PushWithRetry` calls `push` up to `pushMaxAttempts` times in total, sleeping `pushBaseBackoff` before the second attempt and doubling each later delay, only while the error is retryable for `ref`.
A nil `sleep` is the production sleep: the delay plus up to half of it as jitter (the body of fabricengine's `pushSeam.sleepFor`, which card 2 deletes).
Success returns nil; a non-retryable error returns unchanged on its attempt; once the attempts run out it returns an error wrapping both `ErrPushRetriesExhausted` and the last push error, never `ErrPushRejected`.
Each retried refusal logs one Warn naming the ref, the attempt and the stderr line.

`Repo` gains an unexported `pushSleep func(time.Duration)` field, nil in every handle `New` returns, which the three primitives pass to `PushWithRetry` as its `sleep`, so a test in package `gitrepo` shortens the backoff instead of waiting it out.

`Repo.pushWithRebaseRetry`, which `Push` and `PushCoalesced` share, runs its first push and its post-rebase push through `PushWithRetry` with the checked-out branch as `ref` (read with `CurrentBranch`; a read failure leaves `ref` empty, so nothing is retried).
An exhausted error is returned as a plain push failure before `rebaseRetryTriggers` is consulted, so a retryable refusal never triggers `pull --rebase`; this ordering is required because the substring `rejected` in `rebaseRetryTriggers` matches every `! [remote rejected]` line.
`Repo.PushRebaseFree` runs its push the same way, and an exhausted error is returned wrapped as `gitrepo: git push: ...`, never mapped to `ErrPushRejected`.
`DeleteRemoteBranch`, `DeleteRemoteBranchLeased` and `UpdateRemoteBranchLeased` stay unretried.
The retry runs inside whatever lock the caller holds and never rebases or forces, so the push-both-sides shape is unchanged (PATTERN-push-both-sides).

`internal/gitrepo/doc.go` names the retry beside the push surface: which pushes retry, the two refusal forms, the attempt bound and that an exhausted push wraps `ErrPushRetriesExhausted` and never `ErrPushRejected`.

What the retry can now let through: a push the remote refuses with either form is tried again, up to `pushMaxAttempts` in total under the caller's lock; a permanent remote `cannot lock ref`, such as a conflicting ref name, costs the backoff before it fails.

Tests, in `internal/gitrepo/pushretry_test.go`, in package `gitrepo` so they read `pushMaxAttempts` and `retryablePushRefusal`:
a table test of `retryablePushRefusal` over a bare `(failed)`, a remote `cannot lock ref` with a spaced reason, both on the pushed ref in short and full form, the same refusals on another ref's line, a local `cannot lock ref`, a hook-declined refusal, a non-fast-forward `! [rejected]` line and a plain error;
and a table test of `PushWithRetry` with a fake push and a recording sleep: success first time, a refusal then success (one sleep), a persistent refusal stopping at `pushMaxAttempts` pushes with the doubling sleeps and an error satisfying `errors.Is` for both `ErrPushRetriesExhausted` and the last push error and not `ErrPushRejected`, and a non-retryable error returned unchanged after one push.
The ordering at the primitives, which the fake push cannot reach, is pinned in `internal/gitrepo/pushretry_integration_test.go`, tagged `integration` and in package `gitrepo` so it sets `pushSleep` to a recording sleep:
its fixture is a bare remote whose `proc-receive` hook, enabled by `receive.procReceiveRefs=refs/heads`, answers every pushed branch `ng <ref> cannot lock ref '<ref>': ...` and appends one line per invocation to a log, which a local remote gives deterministically where a ref-name conflict such as `x/y` against `x` is refused as `(refname conflict)` and so is not retryable;
one table test over `Push` and `PushRebaseFree` from a clone with a new commit asserts each returns an error satisfying `errors.Is` for `ErrPushRetriesExhausted` and not for `ErrPushRejected`, that the hook ran `pushMaxAttempts` times and the sleep recorded the doubling delays, and that the clone ran no `pull --rebase`: its HEAD is unchanged and no rebase is in progress.
The three new top-level tests raise `internal/gitrepo`'s row in `cmd/lyx/testdata/test-budget.yaml`.

**ImpactSummary:** the three plain push primitives gain a bounded retry; leased and delete pushes and every caller's API are unchanged.

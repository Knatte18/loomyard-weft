```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens over the push-outside-lock diff; one case run per changed function or branch.

## Per-function list

- **`pushPlainWithRetry`, default push on a detached HEAD.** `CurrentBranch` fails and its error returns raw. `mapPlainPushError` wraps it as "git push: …", so it is correct. Before the diff the error was dropped and `git push` failed instead.
- **`pushPlainWithRetry`, `source` is a missing ref.** `ResolveSHA` fails before any push, so no transfer runs and no lock is taken. The error is no longer a `*GitError`, which `mapPlainPushError` still handles. Correct.
- **`pushPlainWithRetry`, `PushCommit` with a SHA source.** `ResolveRevision` resolves the SHA, and the transfer and record follow. Correct.
- **`pushPlainWithRetry`, a commit added while the transfer runs.** The recorded sha is the one read before the push. The tracking ref ends one commit behind origin, and the next fetch corrects it. This is the accepted design decision and is correct.
- **`pushPlainWithRetry`, retries exhausted or push rejected.** The error returns before `recordPush`, so the tracking ref and upstream are untouched. This matches the remote.
- **`resolvePushDestination`, empty remote, branch with no config.** The destination is `origin`, `setUpstream` is true, and the tracking ref is `refs/remotes/origin/&lt;branch&gt;`. `get-url --push` supplies the URL. Correct.
- **`resolvePushDestination`, `branch.&lt;b&gt;.pushRemote` or `remote.pushDefault` set.** The order matches git's. Correct.
- **`resolvePushDestination`, remote not in the repo's local config.** See Finding 2.
- **`recordPush`, nothing to record.** It returns without taking the lock. Correct.
- **`recordPush`, first push.** The tracking ref moves, then both config keys are written under one lock hold. Correct.
- **`recordPush`, failure partway.** See Finding 1.
- **`moveTrackingRef`, delete with the ref absent.** It returns nil. Correct.
- **`moveTrackingRef`, delete with the ref present.** `update-ref -m … -d` runs. Correct.
- **`moveTrackingRef`, create with the ref absent.** `update-ref` creates it. Correct.
- **`moveTrackingRef`, ref already at sha.** It returns nil. Correct, except for Finding 3.
- **`moveTrackingRef`, `keepNewer` and the ref is a descendant of sha.** The ref stays. Correct.
- **`moveTrackingRef`, `keepNewer` and the ref is not a descendant.** It overwrites with sha. This is acceptable: origin held sha right after the push, and the next fetch fixes the ref.
- **`DeleteRemoteBranch`, success.** Correct (see Finding 1 for a local-step failure).
- **`DeleteRemoteBranch`, absent remote ref.** It returns `(false, nil)` and leaves any stale tracking ref, as before the diff. Correct.
- **`DeleteRemoteBranchLeased`, stale lease.** The mapping to `ErrLeaseRejected` is unchanged. Correct.
- **`UpdateRemoteBranchLeased`, a lease backward to an ancestor sha.** The tracking ref is written unconditionally to sha. Correct.
- **`pushWithRebaseRetry`.** The second attempt re-resolves the destination and sha after the rebase. A `recordPush` error is not mistaken for a rebase trigger, because update-ref and merge-base stderr hold none of the trigger strings. Correct.
- **Lock re-entrancy.** `recordPush` takes the lock once, and `moveTrackingRef` calls only `RefSHA`, `IsAncestor` and `config`, none of which take it. Callers of `HoldRemoteRefs` run no push. Correct.

## Findings

1. **LOW · design · `internal/gitrepo/pushdestination.go:95-110`, `push.go:268`, `push.go:171`, `internal/fabricengine/destroy.go:1523-1527`.**
   - A local failure after a landed push is reported as a failed push. The failures are lock acquisition, `update-ref`, `IsAncestor` or `git config`.
   - A default push then returns an error, and `pushClassified` records it as a failed push even though origin holds the commit.
   - `DeleteRemoteBranch` returns `deleted=true` with the error. Its caller `deleteRemoteBranch` records `KindRemoteBranchDeleted` only when `err == nil &amp;&amp; deleted`. The remote deletion goes unrecorded, and a retry then reads "remote ref does not exist" and returns `(false, nil)`.
   - Partial state is also possible: the tracking ref moves and the config write then fails.

2. **LOW · design · `internal/gitrepo/pushdestination.go:37-63`.**
   - The destination is read with go-git `repo.Config()`, which sees only the repository's own config file, not global, system or included config.
   - A remote defined only there reads as "not configured", so `dest.url = dest.remote` and the push goes by name. Git then writes the tracking ref during the transfer, outside the lock.
   - A `remote.pushDefault` set only in global config is ignored and `origin` is used. This is edge-case only.

3. **LOW · design · `internal/gitrepo/pushdestination.go:121-123` against `internal/fabricengine/reflog.go:34-57`.**
   - When `ReconcileOrigin` has already fetched the pushed commit, the early `current == sha` return writes no "update by push" reflog entry.
   - `trackingReflog` stops at the last push entry, so that branch's reflog then has no push boundary. It returns every earlier value, which widens what `classifyRewrite` and the "unpushed" test treat as held.
   - The same happens when `keepNewer` keeps a fetched descendant. The effect is limited to rewind and rewrite classification after a race.

Out of lens, for the handler: `trackingReflogMessage` (`internal/gitrepo/pushdestination.go:19`) repeats the literal that `pushReflogSubject` (`internal/fabricengine/reflog.go:16`) declares. That bears on `PATTERN-named-constants`.

## Tool calls

1. `sed -n 325,375p internal/gitrepo/merge.go` plus a grep for `validSHA`: my context lacked the `ResolveSHA` and `validSHA` definitions.

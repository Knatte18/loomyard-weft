```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals that the fix carried out (F1, F2, F4, F5, F6).

Proposals checked:
- **F1, F4, F5, F6:** I found no case where the proposal is wrong or breaks something.
- **F2:** two findings below. Re-running the wrapped lookups and fetches looks safe: each error stays a `*GitError` through `%w`, and `checkout` and reconcile still read origin once.

1. **LOW. Changed text: `internal/battencli/wire.go`, the `beamForkParent` closure `fabricengine.AddOptions{}.RetryOriginReadInTransit("beam", …)`.**
   - **Summary:** the proposed wrap adds a retry delay at this site that nothing can inject or shorten.
   - **Evidence:** the zero `AddOptions{}` leaves `BackoffSleep` nil, so the method falls back to `time.Sleep` for 1, 2 and 4 seconds. A test of Worktree-Create for a beam member with an unreachable origin would have to wait the real seven seconds.
   - **Rule broken:** the testing rule that a production retry delay is injectable. The review's `Rejected` section accepted the package-level delay list only because the wait goes through a `BackoffSleep` field that a caller can set, and no caller can set it here.

2. **LOW. Changed text: `_lym/landing/summary.md`, "The same retry … covers Add's later origin reads: …".**
   - **Summary:** the sentence claims all of `Add`'s later origin reads, but one is still read only once.
   - **Evidence:** `add.go`, in the `opts.AdoptLocal &amp;&amp; warpAdopted &amp;&amp; !opts.AdoptOnly` branch, calls `warpDefaultBranch`. When `origin/HEAD` is unset, that goes to `remoteDefaultBranch`, an `ls-remote --symref origin HEAD` with no retry (`addadopt.go:60-68`, `cleanupremotewarp.go:349`).
   - **Reach:** Worktree-Create never sets `AdoptLocal`, so only `lym fabric add --adopt-local` gets there. The new `Add` doc line, "each probe, lookup and fetch of a branch the pair is made from", can be read as excluding this lookup. The summary sentence cannot.

Tool calls: none.

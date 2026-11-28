```yaml
lens: fix-code-correctness
outcome: reported
```

Task: correctness check (fix-code-correctness lens) of the code this fix round changed.

1. **LOW**: `_lym/landing/summary.md`, "The same retry … covers Add's later origin reads", and the Add doc in `internal/fabricengine/add.go`.
   - One origin read in Add is still tried only once: `warpDefaultBranch(l, opts)` (`add.go`, under `opts.AdoptLocal &amp;&amp; warpAdopted &amp;&amp; !opts.AdoptOnly`).
   - When `symbolic-ref refs/remotes/origin/HEAD` fails, it calls `remoteDefaultBranch`, which runs `git ls-remote --symref origin HEAD` (`addadopt.go:60-68`, `cleanupremotewarp.go:349-352`). A connection reset there fails Add on the first attempt.
   - The summary's wording reads as covering all of Add's later origin reads. The new doc sentence ("each probe, lookup and fetch of a branch the pair is made from") is arguably narrow enough to exclude this one.
   - Worktree-Create never sets `AdoptLocal`, so the run path the review was about is unaffected.
   - Either wrap that call or name it as not retried.

2. **LOW**: `_lym/landing/summary.md`, "A retried read adds seven seconds of backoff plus up to three more attempts, each bounded by git's remote deadline".
   - An attempt is not always one git call. The weft probe can run `ls-remote`, a tag `ls-remote` and a `fetch`, and each has its own deadline. `TaskBranchOnOrigin` can run several `ls-remote` calls.
   - So one attempt can last several deadlines, and the sentence states a tighter bound than the code holds.

The code paths themselves held up when traced:
- **Error types:** the `FetchWithArgs`, `remoteHeadTip` and `remoteBranchTips` errors all keep the `*gitexec.GitError`, so the new retries do fire on a transport failure.
- **Retry safety:** the fetch refspecs are safe to run again, and a refusal that is not a transport failure returns at once.
- **Callers:** `checkout` and reconcile pass `readOriginOnce`, so their behaviour is unchanged, and both changed packages passed their tests.
- **New test assertions:** the `primeShuttle` log assertion would fail against the old unconditional `taskWorktreeLocation` warning. `OriginNotARepository` would fail if every error were retried.
- **Race:** `restoreErr` has no data race, since Add waits for its probe goroutines before returning.

The new retries at the fetch and beam-lookup sites have no test. The fix added none for them, so I don't count that as a finding.

Tool calls made: none.

```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens on the remote-refs lock diff.

**Per function: case run, verdict**

- `HoldRemoteRefs` (`remoterefslock.go:19`): CommonDir fails → wrapped, nil release. Lock open fails → wrapped. Linked worktree → common dir shared, one lock. Correct.
- `withRemoteRefs` (`remoterefslock.go:32`): lock fails → call never runs and the error returns. call errors → released by defer, error returned. Release error is dropped, acceptable. Correct.
- `FetchWithArgs` (`remoterefslock.go:43`): empty args runs a bare `git fetch`. Lock failure returns `""` plus the lock error. The closure writes the named `stdout` and `err`, and the outer assignment re-assigns the same `err`. Correct.
- `Pull`/`Fetch` (`pull.go:20-45`): git exits 1 → `code`=1, `runErr` nil, the code-handling branch below runs as before. Lock failure → `err` non-nil, `code` stays 0 and is never read, and the wrap is the same shape as a spawn failure. Correct.
- `pushWithRebaseRetry` rebase (`push.go:96`): lock failure → `rebaseErr` is not a `*GitError`, so it returns without `rebase --abort`, which is right because git never ran. Rebase fails → the abort path is unchanged. `rebase --abort` runs outside the lock, but it writes no remote ref. Correct.
- `pushPlainWithRetry` (`push.go:140`) with `PushWithRetry` (`pushretry.go:92-110`): a lock-failure error is not retryable, so `PushWithRetry` returns it at once and the backoff `sleep` runs outside the lock. A git refusal is retried with a fresh lock per attempt. Correct.
- `DeleteRemoteBranch`, `DeleteRemoteBranchLeased`, `UpdateRemoteBranchLeased` (`push.go:193`, `219`, `240`): the inner `err` is local to the closure and the outer `err` receives the returned value. A `*GitError` still reaches the existing classification. A lock failure is not a `*GitError`, so it takes the generic error path. Correct.
- `PushCoalesced` (`push.go:268`): the push lock is taken, then the remote-refs lock inside the push, always in that order. Nothing takes them reversed, so no deadlock. Correct.
- `judgeAgainstOrigin` (`originreconcile.go:744`), two-lock loop: `release` is declared per iteration, so each defer captures its own release. Defers run in LIFO order at return, after the last verdict, so the records lock is released first, then the code lock. The inner `err` shadow is harmless because every return is explicit. Correct.
- `judgeAgainstOrigin`, transport failure: returns `nil, true, nil` and both locks are released. `ReconcileOrigin` returns `res` with `Unreachable` and a nil error. Correct.
- `judgeAgainstOrigin`, reentrancy: `fetchMovedHeads` calls `gitexec.Run` directly. `lsRemoteHeads` calls `gitexec.RunWithin`. `judgeBranch` calls only `gitrepo` reads. None reaches `Fetch`, `Pull` or a push method, so there is no self-deadlock.
- `ReconcileOrigin` (`originreconcile.go:700`): `err != nil || Unreachable` returns early. Correct.
- `fabricengine` fetch conversions (`add.go:415`, `add.go:469`, `archive.go:160`, `checkout.go:216`, `cleanupremotewarp.go:99`, `remove.go:706`, `taskbranchdiscard.go:52`, `weftbranch.go:48`, `remoteonly.go:37`): the dir and args are unchanged. `f.code` is `gitrepo.New(warpPath)` (`fabric.go:77`), the same dir as before. Error type is still `*GitError` via `runChecked`. Correct.

**Findings**

1. LOW, design. `judgeAgainstOrigin` (`originreconcile.go:747-753`, `lsRemotePair` at `~455`): both locks are held across `lsRemotePair`'s retry loop, including `opts.sleep()` between attempts. An unreachable origin therefore blocks every fetch and push of both repositories for the whole retry bound, and those calls would likely fail the same way. Impact is bounded by the retry bound. Evidence: the `defer` of `release` sits before `lsRemotePair`.

2. LOW, consistency. `push.go:153`: `PushRebaseFree` is still documented "Lock-free." It now takes the remote-refs lock on every attempt through `pushPlainWithRetry`. The doc contradicts the code and the new file header at `push.go:2`.

3. LOW, design. `judgeAgainstOrigin` named result `branches` (`originreconcile.go:~790`, `ReconcileOrigin` at `~700`): on a `judgeBranch` error the caller now gets `res.Branches == nil`, where before it kept the verdicts built so far. This only matters if a caller reads `res` after a non-nil error.

4. LOW, design (pre-existing, not worsened). `archive.go:160-163`: `FetchWithArgs` releases the lock before `rev-parse FETCH_HEAD^{commit}`. Another locked fetch in the same worktree can rewrite `FETCH_HEAD` in that window. The unlocked `--refmap=` fetches in `remoteleftover.go:110` and `remoteleftover.go:140` also write `FETCH_HEAD`.

5. NIT, design, unverified. `judgeAgainstOrigin` takes `warpDir` and then `recordsDir` as two `flock`s on separate descriptors. If the two directories ever shared one common git dir, the second acquisition would block forever on the first. I assumed they are distinct repositories and did not check `originTargets`.

**Tool calls**
1. `grep` in `fabric.go` and `pushretry.go`: confirmed `f.code` is `gitrepo.New(warpPath)` and read `PushWithRetry`'s retry classification, neither of which my context had.

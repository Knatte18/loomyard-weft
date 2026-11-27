```yaml
lens: code-correctness
outcome: reported
```

Lens: code-correctness. For each function and branch the diff adds or changes, I ran one concrete case through the code and judged the result.

**Per function or branch: the case run, and whether it is correct**

1. `AddOptions.retryOriginProbeInTransit` (`internal/fabricengine/add.go:88-103`)
   - **`BackoffSleep` nil:** it falls back to `time.Sleep`. Correct.
   - **First probe returns nil:** in loop iteration 0, `IsTransportFailure(nil)` is false, so it returns nil with no sleep. Correct.
   - **First probe fails with a non-transport error** (an auth failure, or `originNamespaceAt` failing locally): it returns at once with the same error. Correct.
   - **Transport failure, then success:** one sleep, then nil. Correct.
   - **Four transport failures:** three sleeps (1 s, 2 s, 4 s), four probe calls, and the fourth error is returned. That matches the doc. Correct.
2. **Warp probe closure** (`add.go:292-296`)
   - **Transport failure, then success:** `warpTip` is reassigned on every attempt, so the final value is the last attempt's. On the final failure `warpTip` is "" and `warpTipErr` is set. Add reads `warpTip` only when `warpTipErr == nil` (`add.go:485-489`). Correct.
   - **Error form:** `remoteHeadTip` wraps the `ls-remote` `*GitError` with `%w` (`remoteleftover.go:255`), so `errors.As` reaches it. Correct.
3. **Weft probe closure** (`add.go:302-306`)
   - **Error forms:** I traced each error path of `probeWeftLeftover`. The `ls-remote` of the branch is wrapped with `%w` (`remoteleftover.go:255`). The `ls-remote` of the archive tags is wrapped with `%w` (`:270`, returned raw at `:286-288`). Both fetches are wrapped with `%w` (`:155`, `:185`). Each of these classifies. Correct.
   - **Swallowed errors:** `isAncestorOrEqual` swallows a `*GitError` from `cat-file`, a local command that `IsTransportFailure` never classifies anyway (`:317-321`). Nothing remote is folded into the probe result. Correct.
   - **Re-run safety:** the re-run writes only `FETCH_HEAD` in the records repo, while the warp probe runs in the warp repo, a different `.git`. That holds the same way the probes already ran in parallel before the diff. Correct.
   - **`SkipGit` / `SkipPush`:** `weftLiveness` returns nil at once, with no retry. Correct.
4. **Add's early-refusal paths with origin down** (`add.go:309`, `defer probes.Wait()`)
   - **Case:** a dirty source, `ErrBranchExists`, or a `Hold` error. Each return now waits out the full probe backoff (7 s of sleep plus up to four attempts per probe) before the refusal comes back, with the teardown lock held throughout. Behaviour is correct and documented ("every return waits for them"); the added latency is part of finding 1 below.
5. **Hold coverage under retry** (`add.go:311-316` together with `:472`)
   - **Case:** a probe whose attempts end on the remote deadline. `IsTransportFailure` counts a non-zero `Timeout` as a transport failure (`transport.go:84`). So each of the three re-runs can last a full remote deadline or stall bound (`stallBound = 30s`, `internal/gitexec/gitexec.go:47`), all under the prime hold. That is not "at most seven seconds", which the summary claims. Finding 1.
6. **`primeShuttle` usage-recorder wiring** (`internal/battencli/drive.go:536-546`)
   - **Absent pair** (`present=false`, `err=nil`): no recorder and no log. Correct.
   - **`taskWorktreePresent` stat error:** `err` is set and logged. Correct.
   - **Pair present but `taskWorktreeLocation` fails:** `err` is set and logged. Correct.
   - **Pair present and located:** the recorder is set. Correct.
   - **Directory present but not resolving as a worktree:** `taskWorktreePresent` returns false, so the case stays quiet. That is consistent with the absent case. Correct.
7. **`TestAdd_RetriesOriginProbesThatFailInTransit`, `OriginAnswersAgain` row** (`internal/fabricengine/add_remoteleftover_integration_test.go:141-150`)
   - **Flow:** both probes fail and call the hook. `sync.Once` restores the URLs, and the second caller blocks until the restore is done. The re-probes succeed. Correct in outcome.
   - **Failure path:** the hook calls `gitkit.Git`, which calls `tb.Fatalf` on a git failure (`internal/gitkit/gitkit.go:64`). The hook runs inside Add's probe goroutines, not the test goroutine. Finding 2.
8. **Same test, `OriginStaysDown` row**
   - The local branch `transient` is absent, so `probeWarp` is true. Six sleeps, a transport failure, and nothing created. Correct.

**Findings**

1. **LOW, design.** `internal/fabricengine/add.go:94-101`, with the summary's claim at `_lym/landing/summary.md:24`.
   - **Summary:** a timed-out origin probe is retried as a transport failure, so the retries can extend the prime hold by up to three remote deadlines on top of the 7 s of backoff, not "at most seven seconds".
   - **Evidence:** `internal/gitexec/transport.go:84` (a non-zero `Timeout` counts as a transport failure) and `internal/gitexec/gitexec.go:47` (`stallBound = 30 * time.Second`).
   - **Same cost elsewhere:** a refused Add waits all of this out before it returns its refusal (`add.go:309`).
2. **LOW, design.** `internal/fabricengine/add_remoteleftover_integration_test.go:141-148`.
   - **Summary:** the `BackoffSleep` hook calls `gitkit.Git` from Add's probe goroutine. On a git failure that calls `t.Fatalf` off the test goroutine, which the testing package forbids, so the failure is misreported or the test hangs instead of failing cleanly.
   - **Evidence:** `internal/gitkit/gitkit.go:64`, and the probe goroutines at `internal/fabricengine/add.go:290-306`.

Tool calls:
1. A grep of `internal/gitexec` for its remote deadline and stall bound, of `wire.go` for `locateTask` uses, and of `gitkit.Git`'s failure mode. The manifest held neither gitexec's timeout values nor `gitkit.Git`'s body.

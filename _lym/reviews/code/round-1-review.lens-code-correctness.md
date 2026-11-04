```yaml
lens: code-correctness
outcome: reported
```

**Task: code-correctness lens. For each changed function or branch I traced one case and give a verdict.**

**List**

- **`Add`, hold taken before any mutation, SkipGit and SkipPush set** (the `add_hold_test` case): no probes start, so `probes.Wait` returns at once, and the hold is released on return. Correct.
- **`Add`, probes started, `Hold` returns an error:** the defers run LIFO. `releaseHold` was never registered, `probes.Wait` runs, then `releaseTeardown`. The error is returned as is before any mutation. Correct.
- **`Add`, early refusal while holding (`ErrSourceDirty`, `ErrBranchExists`, `ErrTaskPathTaken`):** `releaseHold` runs first, then `probes.Wait`, then `releaseTeardown`. The prime lock is released before the network wait, and no probe outlives `Add`. Correct.
- **`Add`, probe data races:** the goroutines write `weftProbe`, `weftProbeErr`, `warpTip` and `warpTipErr`. Every read comes after `probes.Wait()` at `add.go:386`, so there is a happens-before edge. `l` is never mutated before that line; the adopt path copies it (`adoptedAt := *l`). Correct.
- **`Add`, error precedence:**
  - The order is `weftProbeErr`, then the `AdoptOnly` not-live check, then `warpTipErr`, then `judgeWarpLeftover`. This matches the old order, where the warp read was joined after the weft probe, and the weft error was checked after `warpRead.Wait()`. Correct.
  - With `probeOrigins` false, `weftOld = {live: weftBranchAlreadyExists}` as before. Correct.
- **`Add`, `weftBranchExists` and `BranchExists` read before the hold:** the pair's slug is protected by its exclusive teardown lock, and no other create writes that slug's branches. The values cannot go stale through a concurrent create. Correct.
- **`Add`, a refusal now waits on the network:** the probes start before the refusals, so a refusal such as `ErrSourceDirty` returns only after `probes.Wait`. This also happens when the weft repo is missing. It is a latency cost with no wrong result, because the lock is already released. Correct, with a note.
- **`HoldPrimeLock` in `Call`, busy wait ran out, via `Add`:** `Add` returns the `*PrimeLockBusyError` as is. `createRefusal` passes unknown errors through verbatim (`wire.go:380`), so `errors.As` finds it. The row stands Stuck without a halt. Correct.
- **`HoldPrimeLock` in `Call`, lock failure and cancel during the wait:** `primeLockFailure` wraps the `acquirePrimeLock` error, and `Call` unwraps it to `failure.err`. A cancel is checked first through `cancelErr`. Correct.
- **`holds` slice, hold, early release, hold again:** each hold gets its own `sync.Once` wrapper, and the deferred drain releases each once. `Add` releases through the wrapper, so the later drain does not double-release. Correct. A second hold taken while the first is still unreleased would wait on the same flock until the 10 min bound, so the closure must release first. No current path does this.
- **`refuseHeldMember`:** the hold is taken only when `held` is true, and `defer release()` covers every return after it. Lock order there is prime lock, then teardown lock (through `RemovePair`), the same as `Worktree-Teardown`. Correct.
- **`refuseHeldMember`, busy after `Add` succeeded:** the pair stays, the row is Stuck, and the resume path re-judges through the complete-pair branch. Correct.

**Findings**

1. **LOW / design, `internal/fabricengine/add.go:442, 503, 661`.**
   - **What:** network calls still run under the hold, because it is held to `Add`'s return. They are the warp adopt fetch (`FetchWithArgs` at 442), the weft fast-forward fetch (503) and `deleteRemoteBranch` for an archived leftover (661).
   - **Why it matters:** the doc comments, `docs/overview.md` and the commit message say the hold spans no network call, but this is true only of the probes. An adopt, fast-forward or archived-leftover create still queues other creates behind origin.
   - **Evidence:** `add.go:140` and the hold lines at `add.go:234-240`, against the calls above.

2. **LOW / design, `internal/fabricengine/add.go:186-240`.**
   - **What:** `Add` now takes the teardown lock of its slug and then the prime lock, while `Worktree-Teardown` takes the prime lock and then the slug's teardown lock. Before this diff, create held the prime lock first and the order was the same everywhere.
   - **Why it matters:** the argument that this cannot deadlock rests on a create and a teardown of the same slug never overlapping, which nothing enforces here. If they overlap, each waits on the lock the other holds, until the bounded waits expire (`teardownWait`, 10 min).
   - **Evidence:** `teardownlock.go:113-135` shows the bounded wait. `summary.md` states the unenforced premise.

I found no wrong result, swallowed error or leaked lock beyond these two notes.

**Tool calls**
1. Read `internal/battencli/wire.go` lines 374-403. The manifest read stopped at 373, so I did not know whether `createRefusal` returns unknown errors unwrapped.

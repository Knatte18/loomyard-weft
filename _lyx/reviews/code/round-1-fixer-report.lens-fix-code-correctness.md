Fix-code-correctness lens: I ran each changed branch with concrete inputs and found one LOW gap and nothing worse.

**Walkthroughs**
- **`recoveryRetryRead` and `RecoveryRetry`:** the wrapper's `:=` reuses the named results and declares only `err`, so it compiles and behaves as before. For a failed git read, `RecoveryRetry` still returns `(false, recoveries)` and logs at Warn. The `recoveryStartSha` log attribute is dropped, but the wrapped error text carries the SHA. `bs == nil`, `Recoveries == 0`, `Recoveries >= maxRecoveries` and an empty start SHA return the same values as before.
- **`deadEndBatchError`, dead recovery at the cap holding an unrendered amendment:** the first case is false because `!rendersNewAmendment` fails, and the second is false for the same reason, so `why` is empty and the batch is skipped. This is the intended F1 behavior.
- **`deadEndBatchError`, failed git read:** the function returns `fmt.Errorf("%w; way forward: transient, re-run %s", err, reentry)`. The caller at `runlevel.go:617` returns any error unchanged, so `Run` surfaces this plain error without `ErrBatchDeadEnd`.
- **`verifyEveryBatchDone`:** a waiting batch is non-terminal, so it now takes the new `waitsOnParent` case, which comes before `!bs.Terminal`. With a nil `Digest`, `waitingNeed` is empty and the text reads "(waits on its parent: )", which is cosmetic only.
- **`TestRun_DeadEndRefusalSparesRecoverableBatches`:** each subtest builds its own fixture, so the shared `diedMaster` label causes no collision. The dead-with-amendment-at-cap row fails with the old code, because `RecoveryRetry` returned false at the cap and the dead arm refused. The no-counted-recovery dead row, the stuck-at-cap-with-amendment row and the done row pass with and without the fix. They are negative guards against a widened refusal, not regression tests for F1.

**Finding**
- **LOW:** `internal/websterengine/runlevel.go`, the new `err != nil` branch in `deadEndBatchError`. It has no test, so a regression back to treating a git fault as a dead end would pass every test.
  - Evidence: none of the new rows in `runlevel_test.go` has a recorded `RecoveryStartSHA` with a failing git store. The existing two refusal rows use `Recoveries: 1` and no start SHA, so they never reach `committedSince`.
  - The review's F3 asked only for the negative rows, so this is a gap in how F2 was covered rather than a missed instruction.
  - A row with `Recoveries: 1`, a non-empty `RecoveryStartSHA` and a failing fake git would pin the branch. It should assert that the error is not `ErrBatchDeadEnd` and that it names the re-run.

**Tool calls**
1. `grep -n "deadEndBatchError" -B3 -A6 internal/websterengine/runlevel.go`: to confirm how `Run` handles a non-`ErrBatchDeadEnd` error from `deadEndBatchError`, which my context did not show.

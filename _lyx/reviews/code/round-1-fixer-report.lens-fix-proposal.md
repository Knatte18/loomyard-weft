Findings (fix-proposal lens):

1. **MEDIUM** — `deadEndBatchError`, new `recoveryRetryRead` error branch (`return fmt.Errorf("%w; way forward: transient, re-run %s", err, reentry)`), from F2's "pass it up as a plain run-entry error naming a re-run".
   - **Summary:** The proposal labels every git failure "transient". A permanent failure then refuses `Run` on every re-run, with no reset route.
   - **Evidence:** `RecoveryRetry`'s own doc says a start the store does not hold is a git read failure (`committedSince` returns `false, err` when `IsAncestor` errors).
     `fakeGit.IsAncestor` returns `errors.New("fakeGit: unknown commit")` for an unknown SHA.
     The real implementation sits in `isAncestor`, which I did not read, so whether real git errors the same way is unconfirmed.
     Suppose a dead recovery has `RecoveryStartSHA` set to an object that was garbage-collected or lost to a history rewrite.
     Before the fix, `Run` refused with `ErrBatchDeadEnd` and the `reset --to batch-start` way forward.
     Now it returns "transient, re-run", and the re-run fails the same way, so the operator never sees the reset route.
   - **Fix to consider:** Return the error as a plain error but also name the reset way forward, or treat only a failed `HeadSHA` as transient.

2. **LOW** — `runlevel_test.go`, `TestRun_DeadEndRefusalSparesRecoverableBatches` and `TestRun_RefusesBeforeSpawn` (F2/F3).
   - **Summary:** The new git-failure branch has no test, so deleting it would pass every test.
   - **Evidence:** All four new rows leave `Recoveries` at 0 or the cap, so none reaches `committedSince`.
     No existing row seeds `Recoveries: 1` with a `RecoveryStartSHA` the fake git cannot resolve.
     A row like that would exercise the transient-error path.

3. **LOW** — the F1 `rendersNewAmendment(bs)` guard on the dead arm.
   - **Summary:** The guard is broader than F1's "at the cap" scope, but it is correct.
   - **Evidence:** Below the cap, `RecoverSpawnOrAttach` allows a spawn with or without an amendment, so exempting a dead batch that holds an unrendered amendment matches the spawn rule.
     I found no breakage.
   - **Pre-existing, not a finding:** `RecoveryRetry` returns false for `Kind != "recovery"`, so a dead fork-kind batch with zero recoveries is refused as "recovery earns no retry". `recover-batch` could still take it.
     This predates the fix, so I leave it out of scope.

F4 and F6 are mechanical and I found nothing to challenge. For F5, `waitsOnParent` before `!bs.Terminal` is correct because a waiting batch is non-terminal, and `waitingNeed` handles a nil digest.

Tool calls (1 of 6): grepped `IsAncestor` definitions across `internal/`. Context lacked how real git reports a missing start SHA. The grep returned only the `realGit` wrapper and the fake, not `isAncestor` itself.

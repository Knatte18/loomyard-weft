```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, the test that pins it, or none.

**Per behaviour**

1. **`Add` takes `opts.Hold` once and releases it by return on success.** Pinned by `TestAdd_HoldsTheCallersLock/HeldAroundTheAdd` (`add_hold_test.go`). It counts takes and releases only after `Add` returns. It does not check that the lock is held during the mutations, so a hold moved to after the worktree writes would still pass.
2. **A failed `Hold` is returned as is, nothing made, nothing to release.** Pinned by `TestAdd_HoldsTheCallersLock/FailedHoldMakesNothing`.
3. **`Hold` is taken after the probes start and before the fork-point reads.** No test. The new test uses `SkipGit: true`, so `probeOrigins` is false and no probe goroutine runs.
4. **Every `Add` return waits for the probes (`defer probes.Wait`, early refusals and rollback included).** No test. Nothing runs with probes active and a pre-flight refusal or rollback.
5. **Probe errors read only after the pre-flight refusals, and the `AdoptOnly`, `SkipPush` and `SkipGit` behaviour unchanged.** No new test. I did not check whether existing `fabricengine` probe tests cover this precedence.
6. **`Hold` released on a rollback or mid-`Add` failure.** No test. Only success and failed-hold are exercised.
7. **`HoldPrimeLock` wait ran out becomes Stuck without a halt.** Pinned by `TestWorktreeCreate_LockStillHeldPastBoundIsStuck`, which now also asserts `ptr.Halt` is false. The error is returned directly by the closure there, so no test sends a `PrimeLockBusyError` through the real `Add` error path and `createRefusal`.
8. **Acquire failure is a hard error.** Pinned by `TestWorktreeCreate_AcquireError` and `TestWorktreeCreate_AcquireErrorOnLaterAttempt`. The latter checks `errors.Is(err, acquireErr)`, which covers unwrapping `primeLockFailure`.
9. **Cancel during the wait, and cancel before the hold.** Pinned by `TestWorktreeCreate_CancelledDuringLockWait`, `_CancelledContext` and `_CancelledDuringAcquireError`.
10. **A closure that takes no hold never acquires the lock, even while contended.** Pinned by `TestWorktreeCreate_TakesThePrimeLockOnlyThroughItsHold/NoHold`.
11. **A release called twice releases once.** Pinned by `.../ReleasedByTheClosure`.
12. **A hold the closure never releases is released when the row returns.** Pinned by the `released` assertions in `TestWorktreeCreate_CreateErrorsStickWithTheirTextVerbatim` and `_CreatesOnceTheLockIsFree`. The closure there never releases itself.
13. **Several sequential holds in one `Call`, each acquired and released.** No test. `Add`'s hold followed by `refuseHeldMember`'s hold is exactly this case in production.
14. **`refuseHeldMember` takes no lock when the member is not removed.** Pinned by `TestRefuseHeldMember`, whose `holdPrime` fails the test if called.
15. **`refuseHeldMember` takes the hold around the held member's pair removal, releases it, and returns a hold error.** No test. `MemberWhoseBeamHoldsAfterItsPairIsMadeLosesThePair` and `MemberResumedOverACompletePairWhileItsBeamHoldsLosesThePair` pass `holdNothing`, which never asserts a take.
16. **The create closure passes `holdPrime` to `Add`, and the claim, probes, reconcile and `spawnPairPush` run outside the hold.** No test. The `battencli` tests all pass `holdNothing`. Nothing asserts the hold reaches `Add` or that the claim runs before any acquire. This is the behaviour the commit exists for.
17. **Signature change `CreateWorktree(ctx, hold)`.** The compiler pins it. The test edits are mechanical.
18. **Docs and doc comments.** Static prose, exempt.

**Findings**

- **F-A, MEDIUM, design.** `internal/fabricengine/add_hold_test.go:25` (evidence: `add_hold_test.go:49-58`, `add.go:232-240`). Items 3 to 6: no test pins `Hold`'s ordering relative to the probes, that every return waits for the probes, or the error precedence after the move. The only `Add` test runs with `SkipGit`, so the probe path is untouched, and it cannot fail if the hold moves after the mutations. Fix: add a row or assertion with a probing `Add`, where `Hold` observes the probes already started and an early refusal still returns only after the probes finish. Also assert, from inside `Hold`'s release or a mutation seam, that the hold is still held while the worktree is made.
- **F-B, MEDIUM, design.** `internal/battencli/wire.go:941` (evidence: `beamfork_integration_test.go:109,126,181-186`, `wire_seams_test.go:661`). Items 15 and 16: no test asserts the closure hands the hold to `Add`, takes it around a held member's pair removal, runs the claim and `reconcileTaskPair` unlocked, or returns a hold error from `refuseHeldMember`. Fix: in the held-member integration rows, pass a counting hold and assert one take and one release around the removal, and none before the claim. Add a `refuseHeldMember` row where the hold errors.
- **F-C, LOW, scope.** `internal/battenshed/create_test.go:72` (evidence: `holdingCreate`, `create.go:Call`). Items 7 and 13: nothing tests a `PrimeLockBusyError` or `primeLockFailure` wrapped by the closure, as `Add` and `createRefusal` would pass it. Nothing tests two sequential holds in one `Call`. Fix: add a row where the closure wraps the hold error with `fmt.Errorf("%w")`. Add a row where the closure holds, releases and holds again, asserting the acquire and release counts.

I found no duplicate tests. The two new `battenshed` subtests and the two `Add` rows each cover distinct behaviour.

**Tool calls:** none. The diff and all test files were already in context.

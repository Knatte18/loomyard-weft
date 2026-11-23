```yaml
lens: row-validity
outcome: reported
```

Row-validity review of the async-pusher plan's planned test rows. I found 13 defects. I had a six-call budget, so two findings are flagged as unconfirmed where I could not open the code that would settle them.

**BLOCKING**

1. **Card 10: the hand-built teardown receiver panics on the new seams.** Card 10 adds `TeardownDeps.Push` and `Level` and calls them unconditionally in `worktreeTeardownProducer.Call`. Neither the card nor its Intent states what a nil `Push` or `Level` means. `newFakeReceiver` (`internal/battencli/run_test.go:62`, building `Teardown` around line 95) fills only `Shutdown` and `Remove`. Any battencli test that drives the Worktree-Teardown row through it would call a nil func. Card 10 does not list `internal/battencli#newFakeReceiver`, which only card 7 lists, and only to drop `Push`. Fix: state that a nil `Push` pushes nothing and a nil `Level` reads level, or list `newFakeReceiver` under card 10's `Edit` with both seams filled.

**MEDIUM**

2. **Card 10: no fake can show the lock was free during `Push`.** `TestWorktreeTeardown_PushesWithThePrimeLockReleased` asserts "the lock is free during `Push`", and the edited `TestWorktreeTeardown_TearsDownInOrderOnceTheLockIsFree` asserts the order "shutdown, release, push, lock, level, remove". `teardownCallRecorder.calls` (`internal/battenshed/teardown_test.go:21-35`) records only the deps' calls. The lock fakes `waitingPrimeLock` (`create_test.go:47-71`) and `fakePrimeLock` set a single `released` bool and record no acquire or release events into the recorder's sequence. As written, neither row can fail if `Push` runs with the lock held. Fix: list a lock fake that appends "lock" and "release" to `rec.calls`, either an edited `fakePrimeLock` or `waitingPrimeLock`, or a new handle.

3. **Card 10: the new top-level test should be rows of the existing test.** `TestWorktreeTeardown_PushesWithThePrimeLockReleased` checks the same producer sequence that `TestWorktreeTeardown_TearsDownInOrderOnceTheLockIsFree` (`teardown_test.go:238`) and the failure tests already check. Its rows (refused push is Stuck, level false then true, failed `Level`) fit as rows beside `TestWorktreeTeardown_RemoveFailureNamesRemovalHalf` and `TestWorktreeTeardown_ShutdownFailureNeverCallsRemove`. The "release on every exit path" subtest of `TestWorktreeTeardown_PrimeLockDispositions` (`:126-142`) is also not extended to cover a Stuck push path, where the second acquire never happens and the first release must still have run.

4. **Card 3: the prime rows have no fixture to build on.** The card lists `newDivergedPair` under `Uses`, but that helper (`rejectedtakeover_integration_test.go:84-94`) builds one task pair. It makes one `weft-file.txt` commit with no `Run-Transition` trailer and plants a takeover for that pair's slug. The new rows need two batten runs' unpushed commits on the prime's records branch, each carrying its run's `Run-Transition` trailer and staged under that run's directory, with origin moved past them. Without that, `primeTakeovers` finds no slug and no row reaches the read. No such helper is listed. Fix: declare one as a `Create` handle, for example a prime-records fixture.

5. **Card 3: the "takeover read failing" row may not reach the owed branch.** The row plants an unreadable run record on origin's tip, so it depends on `originStatusAtTip` returning an error for a corrupt record. A tolerant read that returns found-false instead would take the no-takeover branch and push. The card names neither the read that fails nor the input that makes it fail.

6. **Card 3: the `IsPushOwed` change is tested through the wrong surface.** Card 3 changes `IsPushOwed` (an error wrapping `ErrTakeoverPassOwed` reads owed, a `*TakeoversError` reads not owed). Neither card 3 nor any other card adds a row to the existing `IsPushOwed` table (`internal/fabricengine/pushowed_test.go:30`). Instead, card 5's `TestRun` adds a row that "reads owed through the real `fabricengine.IsPushOwed`", which tests fabric's predicate from `runpusher`. Fix: move both cases into the `pushowed_test.go` table on card 3, and drop the `TestRun` row; the fake `Owed` seam already covers the retry.

7. **Card 6: `TestPusherDeps` checks only that seams are non-nil.** The card's real contract is not asserted: `Push` passing `OwedUnrecorded`, `InsidePair` true and the slug for a pair, and no slug for the prime. Only `Guard`'s stop through `stoppingPrePushCheck` (`prepush_test.go:381`) can fail if the wiring is wrong. A `Push` wired without `OwedUnrecorded` would pass, and that would make D8 untrue: owed records would come back, and `PushOwed` would hold the run.

8. **Card 8: no row reaches the dobby's pusher restart.** No row reaches the `runpusher.StartIfAhead` call in `DobbyRuns` or in `hubDobbyDeps`' prime `Tick`. The card says `TestStart` and `TestRunCommitsAhead` "hold what it reads", but deleting either call would pass every planned test. The D5 criterion "a dobby tick starts a pusher …" therefore has no row that fails if the tick never starts one.

9. **Card 8: a nil `startPusher` runs real reads in tier0 tests.** In `battenCLI` and `loomCLI`, a nil `startPusher` means "use `runpusher.Start`". So `TestResumeVerb`'s other rows, and every hand-built receiver that resumes (`awaiting_test.go:25` calls `battenPreRun` without `--continue-pass`), would run `TargetOf` → `fabricengine.IsPrimeWorktree` and the start-condition reads on a fake location. That risks git spawns in tier0 files and depends on the fixture. Fix: have a nil seam start nothing in a hand-built receiver, with production wiring filling it, or list the receivers that must set it.

**LOW**

10. **Card 5: the `RolePush` row in `TestEndSubject_EndsTheRunsThenTheSubjectAndItsDobbyLast` likely repeats an existing case.** That test already holds a `RolePush` record (`internal/runproc/end_test.go:480`). Under `runproc.HubOwner`, ending the run slug's processes never reaches a hub record by construction. I did not open the expected-ended list around line 480; if the test already asserts that `RolePush` record is spared, the row adds nothing.

11. **Card 1: the transport row's input may not produce a transport failure (unconfirmed).** It points origin's URL at a missing directory. The existing transport rows use `refusedOriginURL` (`pushclasses_integration_test.go:649`, `677`, `687`), and the missing-directory URL at `:724` serves another case. I could not check how git's "not a repository" error classifies; it may record as `other`, which is a refused class. In that case the row would assert a record that is never written, or would test a refusal. Fix: reuse `refusedOriginURL`.

12. **Card 1: the deferred row only partly adds coverage.** A deferred records push with a record on disk is already reached by `pushanchored_integration_test.go:429-444` and the "BackoffSpent" row at `:586`. The new row's added value is the unrecorded return and the order of the replayed commits; the card should say it asserts the returned error's class and not only that no record exists.

13. **Card 5: the "git or pushes off" row reads process-global environment.** `TestStart`'s row expects "no spawn and no read" with `FABRIC_SKIP_GIT` or `FABRIC_SKIP_PUSH` set. The card does not make the environment read a `starter` seam, so the row has to set env, which conflicts with `t.Parallel` under `PATTERN-test-speed`. Fix: state that the skip flags enter `starter` as a seam.

Tool calls made beyond my context:
1. Grepped fabricengine tests for `newDivergedPair` and the deferral and transport tests, to check card 1 and card 3 fixtures.
2. Grepped the battencli, battenshed, runread and resume tests, to find the receivers and tests cards 7–10 name.
3. Read `continue_pass_test.go`, `newFakeReceiver` and the teardown recorder, to check the seams cards 8 and 10 rely on.
4. Read `TestWorktreeTeardown_PrimeLockDispositions` and `waitingPrimeLock`, to check whether any lock fake records release order.
5. Grepped `stoppingPrePushCheck`, `RolePush` in `end_test.go`, the `battenPreRun` callers and the transport URL fixtures, to check cards 5, 6, 8 and 1.

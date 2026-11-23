```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens, listing for each behaviour the diff adds or changes the test that pins it, or none.

**Per behaviour: pinning test**
- `statuscommit` commits, then starts the pusher once, only when the commit landed: `TestNew_OrdinaryPath`. Pinned.
- `statuscommit` skips while a merge is active: `TestNew_MergeActiveSkips`. Pinned.
- `statuscommit` starts no pusher after a commit error or a re-probe skip: `TestNew_CommitErrorPropagates` and `TestNew_CommitFailureExplainedByTheReProbeTakesTheSkip`. Pinned.
- `statuscommit` panics on a nil `StartPush`: `TestNew_NilPushPanics`. Pinned.
- Batten seam commits and starts the pusher on every row, never pushes, and ignores an old `push_pending`: `TestNewCommitStatusSeam`. Pinned.
- Batten seam board-status write ordering: `TestNewCommitStatusSeam_WritesBoardStatusBeforeRunShed` and `..._BoardWriteFailureDoesNotStopTheCommit`. Pinned.
- Batten seeded Worktree-Create commit goes through the seam with no push: `TestFreshStartCommitsWorktreeCreateThroughTheSeam`. Pinned.
- `commitRunRecords` commits and starts the pusher, with no start for a no-op commit: `TestCommitRunRecords`. Pinned.
- Loom seam commits and starts the pusher: `TestNewCommitStatusSeam_OrdinaryPath`, `_BoardStatus` and `_Finish`, plus `TestCommitStatusSeam_Real` (origin untouched, pusher start false under `go test`). Pinned.
- Loom `commit-records` verb envelope (`committed`, `pusher_started`), error path with no start, and mid-merge: `TestCommitRecordsVerb_Success` and `_ErrorEnvelopes`. Pinned.
- Batten resume starts the pusher before the window reopens, also under `--no-reopen`, and not under `--continue-pass`: `TestBattenPreRun_ResumeStartsThePusher` and `TestBattenPreRun_ContinuePass`. Pinned.
- Loom resume starts the pusher before any driver wake, and not for a done or paused-covered run: `TestResumeVerb` (rows with `ordered`). Pinned.
- Pusher start conditions (lock, refused-push record, pause, hand-over mark, skip env, `IfAhead`, read failures, spawn request carrying `HubOwner` and `RolePush`): `TestStart`. Pinned.
- Pusher body (lock, exit-then-reread, a commit during exit, pause before guard, guard stop, owed backoff 1 s to 60 s, one notice per episode, cancellation, refused push, hand-over): `TestRun`. Pinned. This also exercises `PushUntilLanded`.
- Pusher verb seams (push arguments for pair and prime, guard presence, notify set): `TestPusherDeps`. Pinned.
- Pusher verb resolve error with its way forward, and skipping the persistent pre-run: `TestResolvePersistentPreRun_SkipsSlugGroups`. Pinned.
- `OwedUnrecorded` returns the owed error with no record or notice for a deferred side, a prime transport failure and a teardown-lock hold; a rejection is still recorded: `TestPushPairAnchored_PushesBothSidesRetriesAndReportsEachSide` and `TestPushLock_BoundedWaitGivesUpAndUnboundedWaitBlocks` (the new `OwedUnrecorded_*` rows). Pinned.
- Commit made under a held write lock is pushed in order: `OwedUnrecorded_ACommitMadeUnderTheHeldWriteLockIsPushedInOrder`. Pinned.
- `ErrEntryEnded` for an ended records branch, an ended code branch skipped twice, and the tracking ref dropped with upstream kept: the `OwedUnrecorded_A...EndedEntry...` rows. Pinned.
- `IsPushOwed` for `ErrTakeoverPassOwed` and `TakeoversError`: `TestIsPushOwed`. Pinned.
- Prime takeover read for every run named by an unpushed `Run-Transition` trailer, the `TakeoversError`, an owed failed read, and set-aside of handed-over runs including one at Land: `TestRejectedRecordsPush_ReadsTheTakeover` (new rows). Pinned.
- `HandOverTips` archives the prime tip before the mark, and a refused tag leaves no mark: `TestHandOverTips` (two new rows). Pinned.
- `PushRun` hands over prime runs, returns `ErrHandedOver` when the named run is among them, and owes the pass on a failed archive: `TestPushRun_HandsOverOnATakeover` (three new rows). Pinned.
- Local ahead, refused-push and tips-on-origin reads: `TestRunCommitsAhead`, `TestRefusedPushStands` and `TestTipsOnOrigin`. Pinned.
- `ReadRun` leaves `CodeTipMissing` unset when the local pair holds the tip, and names a failed read: `TestReadRun` (new rows). Pinned.
- Teardown order, the loop back to push when a tip moved, the entry-ended shortcut, the handed-over, refused-push and level-error stucks, and the prime lock released during the push: `TestWorktreeTeardown_*` (four tests). Pinned.
- Teardown over a gone pair finishes Done: `stepTeardown_ResumedRowWhosePairIsGoneFinishesDone`. Pinned.
- Second clone takes over and the unpushed commit stays local: `TestAdoptIntegration_SecondCloneTakesOver`. Pinned.

**Findings**

1. MEDIUM, design. `internal/battencli/wire.go` (the `Teardown.Push` and `Teardown.Level` closures):
   - No test pins the mapping of `fabricengine.ErrEntryEnded` to `battenshed.ErrPushEntryEnded` and of `ErrHandedOver` to `ErrPushHandedOver`.
   - No test pins the real `PushRun` and `TipsOnOrigin` wiring for a present pair.
   - `TestWire_BuildsLazilyForANonexistentTaskWorktree` only checks the closures are non-nil, and the new integration step only covers the gone-pair branch.
   - If the mapping broke, a push of an ended entry would halt the teardown as a refused push, and no test would fail.
   - Evidence: `wire.go` diff lines 1245-1290, `wire_test.go` lines 64-65.

2. MEDIUM, design. The dobby tick and resume restarts have no assertion that a pusher start is attempted.
   - `restartPusher(pair, slug, runpusher.StartIfAhead)` in `internal/battencli/dobbyreads.go`.
   - The prime `RestartPusher` in the `Tick` of `internal/beamcli/dobbydeps.go`.
   - The `startPusher` closures in `internal/battencli/wire.go` (the pair-present branch and its warn branches) and in `internal/loomcli/wiring.go`.
   - Under `go test` `spawnPusher` fails, so the reach-report tests execute these lines but would pass if the calls were deleted.
   - The decision record's D5 acceptance (a tick starts a pusher for a prime or pair ahead, and for none when level or pruned) is pinned only at the `starter.start` level.
   - Fix: inject the start function through a seam and assert a call per tick, or add a fake `starter` row.

3. MEDIUM, design. `internal/runpusher/start.go` and `run.go` real implementations: `tryLock`, `LockPath`, `NewDeps` (`Conditions`, `WorktreeGone`), `pauseCovers`, `handedOver` and `RestartPusher` have no test.
   - `TestRun` and `TestStart` use fakes only, and the package has no coverage map.
   - D3 acceptance "at most one pusher per worktree" and "a pusher whose worktree was removed exits" are pinned only against fake `Lock` and `WorktreeGone`.
   - A broken lock path or a lock that doesn't exclude would pass every test.
   - Fix: add a tier-1 test that takes `tryLock(LockPath(t))` twice for one target and expects the second `acquired == false`, plus a `worktreeGone` row over a temp directory.

4. LOW, design. `internal/fabricengine/pushclassified.go` `checkOrigin`, the `refPruned` branch where the upstream is set, the tracking ref is absent and the branch is still on origin.
   - The push should proceed and skip `holdBackRewound`.
   - The new rows cover only the ended-entry drop, and `pusherreads` rows cover only the local reads.
   - Fix: add a `PushPairAnchored` row with a pruned code tracking ref and a live origin branch, asserting the push lands.

5. LOW, scope. `internal/fabricengine/takeover.go` `primePushTakeovers` and `pushclassified.go` `retryWeftPush`.
   - The `runSlug` appended when no unpushed trailer names it.
   - The `ended != nil` merge of `handedOver` into an origin-derived `EndedSet`.
   - Neither has a row; all prime-takeover rows keep the slug among the unpushed ones and have no ended runs from origin.

6. LOW, consistency. `internal/gitrepo/push.go` `DeleteRemoteTrackingRef`: the documented "absent ref or no refspec mapping is no error" has no direct test, and gitrepo has no test in the diff.

7. NIT, design (test economy). `internal/battencli/commitstatus_test.go`: the `ResumedStartRowCommitsAndStartsThePusher` row of `TestNewCommitStatusSeam` pins the same behaviour as `StartRowsEachCommitAndStartThePusher` and `ChangedPairCommitsAgain`, namely that every row commits and starts. Fold or delete it.

**Out of lens, one sentence each**
- The `slackengine/relay_test.go` change replaces `r.start()` with `r.settle()`. I did not check for a `Loosened-Check:` trailer; the handler should.
- The `runproc.EndSubject` sparing of `RolePush` is not in this diff, and I found no test pinning it.

**Tool calls:** none made. The inherited context already held every diff piece I needed.

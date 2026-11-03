```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, I looked for the test that pins it. I read test function and subtest names and keyword greps, not test bodies. Rows I name as covering a behaviour are inferred from their names.

## Behaviour → pinning test

**gitrepo**
- `PushWithRetryReasons`, the "missing necessary objects" retry, bounded by the existing delays: `TestPushWithRetry_RetriesOnlyRetryableRefusalsUpToTheBound` and `TestRetryablePushRefusal_ClassifiesByTheReasonOnThePushedRefsLine` in `pushretry_test.go`. The end-to-end retry is `RefusedTagPushIsRetriedOnMissingObjects` in `remove_archive_integration_test.go:354`.
- Confinement of the retry to the archive-tag push (`PushWithRetry` passes nil reasons): **none**. No test asserts that a plain push refused with "missing necessary objects" is not retried. LOW.
- `PushBranch`: `TestPushBranch` in `pushbranch_integration_test.go`. Its subtests cover ahead, diverged and local-only refusal (`ErrLocalBranch`). No subtest checks that a branch push moves the tracking ref only under the remote-refs lock, but the lock is not behaviour a unit test can easily observe, and `PATTERN-remote-refs-lock` has its own pinned test.
- `DropRemoteTracking`: `TestDropRemoteTracking`, `pushbranch_integration_test.go:105`.
- `DeleteRemoteBranchLeased` returning `ErrLeaseRejected`: `deleteremotebranch_integration_test.go`.

**Add and adopt-only (D1, D2, D8)**
- Each D1 acceptance bullet maps to an `add_adoptonly_integration_test.go` subtest by name:
  - Worktrees and local branches gone: line 90.
  - Stale registration: line 106.
  - Beam anchor: line 130.
  - Tracked dirt: line 147.
  - Task branch gone: line 164.
  - Weft covered by an archive tag or gone: lines 177 and 189.
  - Local weft diverged from origin's (`ErrRemoteLeftover`): line 250.
  - Reused records worktree, its fast-forward and its rollback: lines 268, 289 and `TestAdd_AdoptOnlyRollbackKeepsReusedRecordsWorktree`.
- Each typed refusal has a subtest:
  - `ErrRecordsWorktreeNotPair`: line 307.
  - `ErrTaskPathTaken`: line 324.
  - `ErrAdoptLocalCheckedOut`, task branch: line 341.
  - `ErrAdoptLocalCheckedOut`, records branch: line 358.
- D8 tracking-ref drop:
  - Weft/warp fork: `TestAdd_DropsStaleTrackingRef`.
  - Adopted local task branch origin lacks: line 381.
  - Local weft branch origin lacks under a stale tracking ref: no subtest by name. It may sit inside `TestAdd_DropsStaleTrackingRef`.
  - "Spawned pair push lands the weft branch on origin": the integration test in battencli (`BranchesLiveOnOrigin`, `RecordsBranchOriginLacks`, `TaskBranchOriginLacks`) most likely carries it.
- Adopt-only skipping every fork-only check: source dirt is pinned by the tracked-dirt subtest. The acting-worktree HEAD read (detached or unborn) and the fork-point empty-marker and anchor-move stop have **no** row. LOW.

**Teardown lock (D7)**
- Add, discard and sync push: `TestTeardownLock_SerialisesAddDiscardAndPairPush`, subtests at `teardownlock_integration_test.go:234-352`.
- Remove versus push: `TestRemove_TeardownLockAgainstThePairsPush`.
- Sweep skip of a held slug: `pairbranch_archive_integration_test.go:133`.
- Sweep live-pair re-read: line 169.
- Sweep lock-error path: line 200.
- Remote-warp sweep: `KeepsTheCandidateWhoseTeardownLockIsHeld` and `KeepsTheCandidateAWorktreeNowHolds`. The remote-warp sweep has **no** row for a lock error other than "held". LOW.
- Synchronous pair push: `TestRunCLI_PushSkippedWhileTeardownLockHeld` for `lym fabric push`. `lym loom commit-records` is pinned in `commitrecords_test.go`.
- `AddLocal` taking no lock: no test.

**Archive and remove (D4, D6)**
- `TestRemove_ArchiveOrder` (`remove_archive_integration_test.go:265`) subtests:
  - Branch pushed before the tag: line 276.
  - Diverged origin tip tagged separately: line 301.
  - Ahead origin tip tagged, no branch push: line 333.
  - Retry on missing objects: line 354.
  - Refused branch push never fails the removal: line 379.
- `TestRemove_LeasedWeftDelete` (`removeremote_integration_test.go:67`) subtests:
  - Origin moved, branch kept and reported: line 77.
  - No origin branch, no delete: line 99.
  - Pushed-tip lease: line 117.
  - Origin-only lease: line 142.
  - Origin-only moved: line 165.
- Sweep: `pairbranch_archive_integration_test.go` has `OriginMovedBeforeTheDeleteKeepsItsBranch` (line 106) and `TagsEveryOrphan...`.
- Sweep with a diverged origin tip archiving both tips: `OriginArchiveTag` appears in that file, so likely covered.
- D4 pause gate on the branch push: **none**. Nothing tests that a covering pause holds `Remove` at the branch push. LOW; the gate is inherited from `withRemoteRefs`.
- `RemoteBranchKeptReason` and `OriginArchiveTag` envelope fields: `removefields_test.go`.

**Reconcile (D1 run scope)**
- The `PairMayBeAbsent` subtest at `originreconcile_integration_test.go:676` covers absent pair, entry verdicts, and refusal without the option.
- `DropLocal` keeping today's error: the without-option refusal covers `originTargets`, but no test goes through `DropLocal`. Covered indirectly.

**battencli resume (D1, D2)**
- `TestOwnerPreflight_GoneTaskWorktree` (`owner_readopt_test.go`) has rows for:
  - Order of reconcile, prune, add.
  - Owned elsewhere, ended and stopped refusals.
  - Unreachable origin, reconcile error, prune error.
  - Transport error, lock wait and failed worktree add as transient.
  - Gone-read error.
  - Present worktree, Worktree-Create and Worktree-Teardown skipping the seams.
  - Cancel naming at a row before Land.
  - Land restore from the newest finished tag, the legacy segment, no finished tag, tag read error, task branch only, and leftover with or without upstream.
  - The four closed-set refusal ways forward.
  - A failed run keeping its state with no notice.
  - A held run lock refusing as transient.
- `TestBattenIntegration_GoneTaskWorktree` covers live branches, `RecordsBranchOriginLacks`, `TaskBranchOriginLacks`, the cancel-naming failed stop, the landed restore and re-adopt, and stray paths read as gone.
- `refuseContinuePass`: the "gone refuses" row is in `continue_pass_test.go:58`. The gone-read **error** path has no row. LOW.
- Cancel-ordering warning about `lym fabric cleanup --apply` between restore and re-run: the Land restore rows probably assert it. Unverified.
- Latest-committer-date choice between several `done` tags, including a diverged-tip tag with no `done` status: the "newest finished tag" row exists. I could not confirm it holds two `done` tags with different committer dates plus a non-done newer tag. If it does not, the tie-break behaviour is unpinned. Unverified.
- `readoptCause` rewording of the checked-out refusal: covered by the checked-out rows if they assert the stop text drops `--adopt-local`. Unverified.
- Failed-stop commit and status push (`CommitFailed`): the `owner_readopt_test.go` fake covers the call. `readopt_integration_test.go` has `armFailedStop`, so a real status write is covered.
- Notice posted once: `Notice` appears in both files.
- Dobby planning no further pass after the failed stop: no test in `passes_test.go` for it. That test file changed by one line only. LOW, since this depends on existing `failed` handling.

**Follow-up wording (D3)**
- `TestBattenFollowUp` (`pairteardown/followup_test.go`) covers no record, done, Worktree-Teardown, owner pause, before-Land cancel, and Land.
- `TestSettleBoardClaim_RunningBattenRecordWayForward` and `TestTeardown_CallSequence` cover the `lym fabric remove` warning (`Result.Warning`).
- Fabric envelope `warning`: `removefields_test.go`.
- `preflightRunOwner` follow-up and `withFollowUp`: `TestPreflightRunOwner_RemoveFollowUp`.
- Cancel refusal: `StoppedBranchNamingTheRemovalAtLandEndsInTheLandWayForwardNotTheCancel` in `cancel_test.go:494`.
- Tender failed end: `TestOwnedElsewhereEnder`.
- Landingshed: `finalize_test.go` (33 added lines), covering the Finalize owner-pause exception (nil follow-up) and the stuck follow-up.
- `loomcli.landingDeps` `RemoveFollowUp` wiring and its unreadable-prime branch: **none** found among the changed tests. A drift guard exists but checks only that the field is non-nil. LOW.

**Records reads**
- `TestSlugRecordsTags` (`recordsreads_integration_test.go`) covers `SlugRecordsTags`, `RecordsBranchFile` and `RecordsBranchHasUpstream`. This includes the no-write guarantee for origin-only tags.

**Stencil D5**: exempt (static prose).

## Findings

None at MEDIUM or above. LOW coverage gaps, all `consistency`:
1. L1: no test that a plain push refused with "missing necessary objects" is not retried. Evidence: `gitrepo/pushretry.go` `PushWithRetry` passes nil reasons, and `pushretry_test.go` has only the two table tests.
2. L2: adopt-only's skip of the acting-worktree HEAD refusal and of the fork-point marker and anchor-move stop is unpinned. Only the dirt skip has a subtest. Evidence: `add.go` `!opts.AdoptOnly` guards, and `add_adoptonly_integration_test.go:90-250`.
3. L3: the remote-warp sweep's non-held lock-error path has no row, though the weft sweep does (`pairbranch_archive_integration_test.go:200`). Evidence: `cleanupremotewarp.go` `deleteLeftoverUnderLock`.
4. L4: the gone-read error branch of `refuseContinuePass` has no row. Evidence: `continue_pass_test.go:58` covers only gone=true.
5. L5: the `loomcli` `landingDeps` `RemoveFollowUp` closure, including its unreadable-prime branch, has no test among the changed tests.

## Tool calls
1. `git diff --name-only | xargs grep` of subtest and name lines: the manifest held no test files. The output was too large and was persisted unread.
2. The same, restricted to `^func Test` names: to list test functions.
3. A keyword grep per behaviour over changed test files: to map behaviours to files.
4. A grep of subtest names in battencli, teardown-lock, archive and reconcile test files: to see which rows exist.
5. A grep of subtest names in the adopt-only, remove, gitrepo and reconcile test files, plus a pause/`ErrLocalBranch` grep: to confirm the D1, D4 and D6 rows.

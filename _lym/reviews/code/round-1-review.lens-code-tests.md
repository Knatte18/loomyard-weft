```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens. For each behaviour the diff adds or changes, I looked for the test row that would fail if that behaviour broke.

## Behaviour → pinning test

1. **`UpdateRemoteBranchLeased` wraps `ErrPushAuthRefused` on an auth refusal.** Pinned indirectly by `TestCloneSingleRepo_FailedSeedPushLeavesNothing/RefusedCredentialRemovesTheHubAndNamesTheCredentials`. Its text "for want of authentication or permission" is reached only through `errors.Is(..., ErrPushAuthRefused)`. No `internal/gitrepo/push_test.go` row drives `UpdateRemoteBranchLeased` itself.
2. **`seedPushOutcomeUnknown` and `seedPushCause` on an auth refusal of the records push.** The same `RefusedCredential...` row pins both. It checks that the hub is removed, that "may hold" is absent, and that origin's namespace is empty.
3. **Auth refusal on the board push, after the records push landed.** No test. The test hook returns at once for any ref other than `recordsRef`.
4. **`RunTeardownChild` keeps an earlier record on empty input.** Pinned twice: by the `NoRemoteStepsKeepsTheEarlierRecord` row of `TestRunTeardownChild_SettlesThePendingRecordOnceItEnds`, and by the new `PendingTeardowns` assertion in `TestRunTeardownChild_HoldsTheLockUntilItsInputCloses`.
5. **`CleanupRemoteWarp` pushes the slug's local archive tags before the delete.** Pinned by `TestCleanupRemoteWarp_ApplyDeletesOnlyTheCandidate/PushesTheLocalArchiveTagThatHoldsTheCandidatesWork`. It fails on the base: origin's tag would be absent.
6. **A failed tag push in the sweep keeps the branch and fills `Error`.** No test.
7. **`existingHubError` for a whole hub names `mv` and no `--reset`.** Pinned by `TestCloneHub_HubExistsCheckPrecedesProbeInTwoArgForm`, new assertion at line 605.
8. **`existingHubError` for a hub without Bolt still names `--reset`.** Pinned by `internal/fabriccli/reconcile_missingbolt_integration_test.go:66`.
9. **`--reset` refuses a hub holding a pair worktree or an unpushed code commit, and keeps the hub.** Pinned by `TestCloneOneURL_JoinsTheNamespacedHub/ResetOfAHubHoldingWorkOriginLacksIsRefusedAndKeepsTheHub`, rows `Pair` and `UnpushedCodeCommit`.
10. **`--reset` still accepts a clean whole hub, so no false refusal.** Pinned by the existing reset successes: `TestCloneHubReset` (`livestate_verbs_test.go` `cloneHubResetRealHubCase`), `warpbinding_clone_integration_test.go:245` and `shortnamebinding_clone_integration_test.go:118/134`.
11. **A commit that a tag holds is not counted as work origin lacks.** No row drives a tag-only commit.
12. **`FetchRecords` refuses a lost records layout.** Pinned by the new `FetchRecords` row of `TestOneRepositoryPairTips_RefuseARecordsStoreThatLostItsFetchRefspecs`.
13. **`fetchRunForCatchUp` uses `FetchRecords` for a run with no pair here.** No test. Reverting `catchup.go:237` to the raw fetch fails nothing.
14. **`SettleGonePairTeardown` reports settled true or false.** Pinned by `TestRemove_PendingRecordNamesTheRerunItsCallerGave`, which calls it three times: false with the pair on origin, true with the pair gone, false with no record.
15. **A removal's records-archive way forward carries no "when origin still holds" condition.** Pinned by the `TestNoticeRemoteTeardown` rows `RemoveArchiveFailureWithWeftTip` and `...AndNoWeftRoot` (`wantAbsent`), and by `TestRemove_RerunAfterItsTeardownChildEndedArchivesTheLocalRecordsTip`.
16. **`remover.run` with `--remote` settles a gone pair's record, under both verdicts, and refuses with no record or on a failed settle.** Pinned by four `TestRemover` rows.
17. **`remover.run` without `--remote` returns `ErrPairNotFound` and never calls settle.** No test. `TestRemover` always calls `r.run(..., "s", true, true)` (`remove_test.go:309`).
18. **The CLI wiring of `settle`, which passes rerun `lym fabric remove <slug> --remote`.** No test.

## Findings

- **MEDIUM, design. `internal/fabricengine/cleanupremotewarp.go:169`.** The sweep's failed tag push, which "keeps the branch and fills the entry's Error", has no test.
  - Nothing fails if the `return` after `entry.Error = err.Error()` is dropped and the delete proceeds.
  - `SetArchiveTagPushHookForTest` (`internal/fabricengine/export_test.go:693`) already gives the seam to refuse the push.
- **MEDIUM, design. `internal/battencli/catchup.go:237`.** The catch-up's no-pair call site has no test that fails on the base code.
  - The `FetchRecords` row in `codetipsettle_integration_test.go:55` drives the new function over a pair's records worktree. Reverting the call site to `gitrepo.New(fabricengine.RecordsWorktree(prime)).Fetch()` fails nothing.
  - The board entry's fault "`fetchRunForCatchUp` reads no records layout" asks for a test that fails first. The summary says the call site was not driven, and blames the per-process layout cache.
- **MEDIUM, design. `internal/fabriccli/remove.go:113`.** The `remote &&` guard is unpinned: nothing fails if it is dropped and settle runs without `--remote`.
  - Every `TestRemover` row runs with `remote = true` (`internal/fabriccli/remove_test.go:309`).
  - So no row shows that `lym fabric remove <slug>` without `--remote` still returns `ErrPairNotFound` and never calls settle.
- **LOW, design. `internal/fabricengine/destroy.go:1806-1820`.** An auth refusal on the board push, after the records push landed, has no test.
  - That path reaches `seedPushCause("board", true, ...)` and `fail(..., recordsTip)`, which withdraws the landed records branch.
  - The `RefusedCredential...` hook at `clone_singlerepo_integration_test.go:248` returns at once for any ref other than `recordsRef`, so only the first push is refused.
- **LOW, design. `internal/fabricengine/clone.go:871`.** The `--tags` exclusion in `hubWorkOriginLacks` is unpinned.
  - No row drives a local commit that only a local tag holds and expects `--reset` to accept it, so dropping `--tags` fails nothing.
  - The existing `--reset` successes run over hubs with no such commit.
- **LOW, consistency. `internal/fabricengine/remoteteardown_test.go:589-592`.** The new assertion duplicates existing coverage.
  - The `PendingTeardowns` assertion added to `TestRunTeardownChild_HoldsTheLockUntilItsInputCloses` covers the same behaviour as the `NoRemoteStepsKeepsTheEarlierRecord` row of `TestRunTeardownChild_SettlesThePendingRecordOnceItEnds` (`pendingteardown_integration_test.go:471-476`).
  - That behaviour is: a child given no job keeps an earlier record. Both tests pin it.
- **LOW, design. `internal/fabriccli/fabric.go:1034-1036`.** The CLI's `settle` wiring has no test: neither the rerun string it passes nor its use of `top` is driven. `TestRemover` replaces `settle` with a stub.

## Tool calls

1. A grep of the test files for the existing-hub messages, the users of `ErrPushAuthRefused`, the archive-tag push hook, and `FetchRecords`/`fetchRunForCatchUp`. The manifest held no test-side search for these behaviours.
2. A read of `internal/fabriccli/remove_test.go:140-330`, for how `TestRemover` sets `remote` and the rows outside the diff hunk.

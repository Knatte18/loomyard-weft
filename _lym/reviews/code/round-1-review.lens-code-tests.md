```yaml
lens: code-tests
outcome: reported
```

Lens: code-tests. For each behaviour the diff adds or changes, I looked for the test row that fails if that behaviour breaks.

**Per behaviour, the test that pins it:**

1. **O1, `RefuseHubWithoutBolt` on a one-repository hub (code URL alone, `--into` the hub's parent, `--reset`).** Pinned by `TestCommand_HubWithoutItsBoltNamesTheResetClone`, which compares the whole message.
2. **O1, `RefuseHubWithoutBolt` on a two-repository hub (records URL, then code URL).** None.
3. **O1, a hub with a Bolt repository passes the refusal.** Pinned by the existing reconcile tests the reach report lists, such as `TestCommand_ReconcileFromAPairAtAnotherAnchor`.
4. **O1, `existingHubError` in the one-URL join form.** Pinned by the `CloneHub` half of `TestCommand_HubWithoutItsBoltNamesTheResetClone`, which compares the whole message.
5. **O1, `existingHubError` in the two-argument form, and `cloneResetCommand`'s `--single-repo`, `--shortname`, `--subpath` and second-URL arguments.** None. `warpbinding_clone_integration_test.go:467` only checks that the error contains "hub already exists".
6. **O2, `resolveMergeSources` refusing an unreadable layout, through `mergeIn`.** Pinned by `TestOneRepositoryMerge_RefusesARecordsStoreThatLostItsFetchRefspecs`.
7. **O2, the same refusal through the `Merge` verb's call site (`merge.go:409`).** None. The function is shared with the pinned path, but this call site's early return is not pinned.
8. **O2, `fastForwardWeft` skipping a weft whose layout cannot be read while the code side still moves.** None. The reach report says no mapped test reaches `fastForwardWeft`, and no `TestFastForwardPair_*` test unsets the refspecs.
9. **O2, `FetchPairTips` and `ReturnToPreviousTransition` returning the layout error with nothing fetched.** None. No test in `codetipsettle_integration_test.go` unsets the refspecs.
10. **O4, `PlainArchiveTagsOnOrigin` listing plain archive and owner-archive tags.** Pinned by `TestOneRepositoryOrigin_PlainArchiveTagsAreListedWithTheirMove`. It covers both prefixes, sorting, the exact `WayForward` string, the exclusion of namespaced tags, and the move's effect.
11. **O4, a two-repository hub or an origin-less code store listing none, and an `ls-remote` failure.** None.
12. **O4, the cleanup envelope's `plain_archive_tags` field, and the cleanup verb failing on a listing error.** None. `TestCommand_CleanupRemoveScenario` reaches the code, but no test reads `plain_archive_tags`; a grep finds only `warp_entries` and `warp_skipped_reason` assertions.
13. **O5, a pending record carrying `Rerun` and printing "run \`<cmd>\` again" after the tag push.** Pinned by `TestDiscardTaskBranch_RerunAfterItsTeardownChildEndedArchivesTheTaskTip`.
14. **O5, a record from an earlier binary with no `Rerun` still getting the description.** Pinned by rows in `TestPendingTeardowns` (`pendingteardown_test.go:59,62`) and `TestNoticeRemoteTeardown` (`remoteteardown_test.go:47`).
15. **O5, the lock refusal naming "re-run \`<cmd>\`" when `rerun` is set.** None. `teardownlock_integration_test.go:342` passes `""` and asserts only `errors.Is(ErrPairPushRunning)`.
16. **O5, `lym batten cancel` passing `"lym batten cancel <slug> --remove"`.** None. The reach report says no mapped test reaches `hubCancelDeps`.
17. **O3 help text and doc comments.** Exempt as static prose.

**Findings:**

- **MEDIUM, design.** Locations: `internal/fabricengine/parenttips.go:410` (`fastForwardWeft`) and `internal/fabricengine/codetipsettle.go:24,44` (`FetchPairTips`, `ReturnToPreviousTransition`).
  The layout reads O2 adds on these three paths have no covering test.
  Only `resolveMergeSources` is pinned, by `onerepository_lifecycle_integration_test.go:41`. The way `TestPull_RecordsStoreWithNoNamespacedFetchRefspecFails` (`pull_integration_test.go:693`) does it would work here: unset `remote.origin.fetch` and call `ForgetOriginNamespaceForTest`.
  Evidence: the reach report ("no mapped test reaches internal/fabricengine.Fabric.fastForwardWeft"), and the `TestFastForwardPair_*` and `TestReturnToPreviousTransition` lists in `parenttips_integration_test.go` and `codetipsettle_integration_test.go`.
- **LOW, design.** Location: `internal/fabricengine/clone.go:111` (`RefuseHubWithoutBolt`) and `clone.go:44` (`cloneResetCommand`).
  The two-repository refusal (records URL, then code URL) and the two-argument `existingHubError` have no test that asserts the command they name.
  Evidence: `warpbinding_clone_integration_test.go:467` checks only the "hub already exists" substring, and the new test drives only a one-repository hub (`reconcile_missingbolt_integration_test.go:109`).
- **LOW, design.** Location: `internal/fabriccli/fabric.go:903-907` and `internal/fabricengine/cleanupremotewarp.go:34-40` (diff lines).
  Three things go unasserted: the `plain_archive_tags` envelope field, the cleanup verb failing when the listing errors, and the empty list on a two-repository hub or an origin-less store.
  Evidence: a grep of `internal/fabriccli/*_test.go` finds no `plain_archive_tags`, and `TestOneRepositoryOrigin_PlainArchiveTagsAreListedWithTheirMove` drives only a one-repository hub with an origin.
- **LOW, design.** Location: `internal/fabricengine/taskbranchdiscard.go:49-52` and `internal/battencli/cancelcmd.go:288`.
  Two O5 pieces have no test: the lock refusal naming the given rerun command, and cancel passing `lym batten cancel <slug> --remove`.
  Evidence: `teardownlock_integration_test.go:342` passes `""` and checks only `errors.Is`, and the reach report says no mapped test reaches `hubCancelDeps`.
- **NIT, design.** Location: `internal/fabriccli/reconcile_missingbolt_integration_test.go:110,121`.
  The expected refusal is built from `fabricengine.BoltDir(h.Path)` and `fabricengine.BoltDirName`, production helpers the code under test also calls, instead of a literal such as `filepath.Join(h.Path, "_bolt")`.

Tool calls:
1. Grepped tests for `plain_archive_tags`, `FetchPairTips`, `FastForwardPair`, the hub-exists and rerun strings, and `ForgetOriginNamespaceForTest`: the manifest did not cover which tests exist for these behaviours.
2. Read `remoteteardown_test.go` 35–80, `pendingteardown_test.go` 40–75, the layout greps in the settle and fast-forward tests, and the cleanup CLI test's envelope assertions: to see what the existing rows assert.
3. Read `pull_integration_test.go` 690–720 and the test lists of `codetipsettle_integration_test.go` and `parenttips_integration_test.go`: to confirm no layout test covers the fast-forward or settle paths.

```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, name the test that would fail if it broke.

**Per behaviour, the test that pins it**

1. **Unpushed records-store commit refused** (one-repository): `TestCloneOneURL_JoinsTheNamespacedHub` row `UnpushedRecordsCommit`. In the two-repository layout, only the plain `--reset` success cases pin the count (see 13).
2. **Unpushed board commit counted in the records store**: row `UnpushedBoardCommit`.
3. **Uncommitted change in the Bolt checkout refused**: row `UncommittedBoltChange`.
4. **Ignored file in the prime refused and named**: row `IgnoredFileInThePrime`.
5. **Ignored files, stash and uncommitted changes in the records store, and ignored files in Bolt**: no row of their own. The checks run per store through `storeWorkOriginLacks`. The prime rows pin each check, and rows 1 and 3 pin that the records and Bolt entries are in the store table. I count this as covered.
6. **Bolt read with `ownsRefs: false`** (no stash or commit count there): none. Removing it changes no observable result, so no finding.
7. **Cap of 5 named ignored paths and the "and N more" count**: none.
8. **Skipping of links, `.lym` and `.weft` in `ignoredWork`**: only incidental. The plain-reset success tests `ResetProceedsOverACommitOriginHoldsUnderATagAlone` and `ResetKeepsRecord` (in `TestCloneHub_ShortnameRecordLifecycle`) pass only if a hubforge hub's wired links and Bolt `.lym` are skipped. No row puts a `.weft` directory, a nested `.lym` or an ignored link in front of the filter and asserts it is skipped.
9. **`DiscardWork` lets `--reset` pass store work**: one-repository by `ResetWithDiscardWorkReplacesAHubHoldingStoreWork`; two-argument form at both anchors by the `TestCloneHubReset` RealHub cells (`cloneHubResetRealHubCase`).
10. **`DiscardWork` never passes a pair or a teardown**: rows `Pair`, `PendingTeardown` and `TeardownStillRunning` with `discardable: false`, which re-run with `DiscardWork` and assert the refusal.
11. **`DiscardWork` without `Reset` refused before anything is created**: `TestCloneHub_DiscardWorkWithoutResetIsRefused`.
12. **The refusal names the `--reset --discard-work` clone only for store work** (`hubWorkWayForward`, `cloneResetCommand`): the `discardable` assertion in the refusal table.
13. **`existingHubError` over an unfinished hub**: `TestCloneHub_HubExistsCheckPrecedesProbeInTwoArgForm` checks both the work-plus-discard-clone message and, after a push, the plain `--reset` clone.
    - Plain `--reset` succeeding over a hub with no work: `ResetProceedsOverACommitOriginHoldsUnderATagAlone` (one-repository) and `ResetKeepsRecord` (two-repository, after `push --all`).
    - `existingHubError` with a pair in an unfinished hub ("land or remove"): none, but that wording is unchanged from the old refusal, so no finding.
14. **Seed refusal names `--reset --discard-work`**: `TestCloneSingleRepo_FailedSeedPushLeavesNothing/BoardWithNoUpstreamIsNeverPushed`.
15. **The `--reset` refusal in `RefuseHubWithoutBolt`, two-repository**: `TestCommand_HubWithoutItsBoltNamesTheResetClone/TwoRepository`.
16. **CLI `--discard-work` flag passed through `cloneFlags` into `CloneOptions.DiscardWork`**: none. A search of every `_test.go` under `internal` and `cmd` finds no test using the flag or `discardWork` outside the engine tests.

**Findings**

- **MEDIUM · design** · `internal/fabriccli/fabric.go:100,115`, `internal/fabriccli/clone.go:56`
  The CLI's pass-through of `--discard-work` has no test. A dropped `GetBool` or a missed `cloneFlags.discardWork` → `CloneOptions.DiscardWork` mapping would pass every test.
  Evidence: diff 001 lines 100, 115 and 56; the search found no CLI test for the flag.
- **MEDIUM · design** · `internal/fabricengine/clone.go:1043-1053`
  The cap of `ignoredPathsNamed` and the "and %d more" count have no test. Only `IgnoredFileInThePrime` exists, with one path. An off-by-one or a lost count would pass.
  Evidence: `clone.go:1043-1053`; `clone_singlerepo_integration_test.go`, row `IgnoredFileInThePrime` (diff 002 lines 499-512).
- **LOW · design** · `internal/fabricengine/clone.go:1089-1094`
  `ignoredWork`'s skips have no row that asserts them. The base-name skip of `.lym` and `.weft` and the link skip are pinned only incidentally, by plain-reset success over fixture hubs. Nothing checks a `.weft` lock directory or an ignored link; a narrowed filter would show only by accident.
  Evidence: `clone.go:1089-1094`; success tests at `clone_singlerepo_integration_test.go:455-471` and `shortnamebinding_clone_integration_test.go:136-139`.

Tool calls:
1. Searched `_test.go` files for `DiscardWorkFlag`, `discard-work`, `discardWork` and `Reset: true`. My context had no CLI or other test listing for the flag or for plain resets.
2. Read `shortnamebinding_clone_integration_test.go:108-145` and `clone_singlerepo_integration_test.go:455-495`, to check that the plain-reset tests assert success. My context had only the hunks the diff shows.

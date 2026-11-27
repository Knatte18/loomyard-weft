```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, name the test that would fail if it broke.

**Per behaviour, the test that pins it**

1. `fabricSync` takes `opts`, and `SkipGit` returns `(false, nil)` before the fabric opens: `TestFabricSync_GuardOrdering` (`internal/webstercli/cli_test.go:75-96`, both rows).
2. `fabricSync` hands `opts.SkipPush` on to the commit: `TestFabricSync_CommitsAtEveryRelPathDepth` (`sync_integration_test.go:184-201`). Without the option, the push to a remote-less records repo fails.
3. `fabricSync` with zero options does real git work: `TestFabricSync_ReportsCommittedWhenCorrespondenceRecordFails` and `TestFabricSyncWayForward_NextSyncCommitsSavedState`.
4. Every verb passes `c.syncOptions` to `fabricSync`: the `newSkipGitVerbsFixture` tests in `verbs_test.go`, `acceptaudit_test.go`, `fixplan_test.go`, `fixverify_test.go` and `secretpaths_test.go`. Their fixtures have no records sibling, so a verb that dropped the field would hit the sync failure. The failing-opener way-forward tests cover the error path.
5. `reset` passes `c.syncOptions` to the closing sync: the `TestResetCmd` subtests (`reset_test.go:83-765`).
6. `reset` passes `c.syncOptions` to `ResetPairCode`: no test pins this. The `newResetFixture` header says the remote update ignores `SkipGit`, so no row would fail if `ResetPairCode` got zero options. This was already uncovered before the change, when it read the environment, so it is not a finding.
7. `masterStrandLookup(name, resolve)`: the subtest at `batchpause_test.go:241-262` covers the empty name, the resolved name and a resolve error.
8. `refuseOrphanedMaster` reads `c.strandName`: `TestRefuseOrphanedMaster_RefusesAMasterWhoseRunEnded` (`ownership_test.go:71`, `:102`).
9. `wireHub` and `wireStandalone` fill `c.syncOptions` from `EnvSyncOptions()` and `c.strandName` from the strand-name variable (`wiring.go:218-220`, `:357-359`): **none**.
10. `wireHub` and `wireStandalone` build `c.masterStrand` from `c.strandName`: **none**. That is the same gap as item 9.
11. loomcli: the diff changes no production code there, so the lens has no behaviour to pin.

**Findings**

- **MEDIUM**, class `design`, at `internal/webstercli/wiring.go:218-220` and `:357-359`: no test checks the wiring-time read of the environment.
  - Before the change, the tests that set `FABRIC_SKIP_GIT` or the strand-name variable exercised the production read as a side effect, because the verbs read the environment at call time.
  - Every such test now sets `syncOptions` or `strandName` directly on the `websterCLI` it builds, so nothing reaches wiring.
  - Deleting `c.syncOptions = fabricengine.EnvSyncOptions()` or `c.strandName = os.Getenv(agentname.StrandNameEnv)` from either wiring path would pass every test. Production would then stop honouring `FABRIC_SKIP_GIT` and `FABRIC_SKIP_PUSH`, and the orphaned-Master guard and the Master claim would stop working.
  - Evidence: a grep of `internal/webstercli/*_test.go` for `StrandNameEnv`, `LYM_STRAND_NAME`, `FABRIC_SKIP`, `syncOptions` and `strandName` outside the fixture setters finds only the `masterStrandLookup` unit subtest.
  - Fix: add assertions to an existing serial `wiring_test.go` case, for both `wireHub` and `wireStandalone`. Set `FABRIC_SKIP_GIT=1`, `FABRIC_SKIP_PUSH=1` and the strand name with `t.Setenv`, then assert `c.syncOptions` equals `{SkipGit: true, SkipPush: true}` and `c.strandName` equals the set name.

Also noticed, outside this lens: the folded `TestWireLanding_FillsTheLandingSeamsPerRecipe` now checks language skills only under the sampler and darn recipes, no longer under the empty default recipe (`wiring_publish_integration_test.go:60-80`).

Tool calls:
1. Grepped the webstercli test files for the strand-name variable, the `FABRIC_SKIP` variables, `syncOptions`, `strandName` and `masterStrand`. The manifest grep left out `wiring_test.go`'s and `cli_integration_test.go`'s use of the strand-name variable and of the `FABRIC_SKIP` variables, so I needed it to confirm the wiring read is uncovered.

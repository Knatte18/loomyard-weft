# Card 3 — portals-unwired

**Edit:**
- `internal/fabricengine#Topology.Add`
- `internal/fabricengine#Topology.AddLocal`
- `internal/fabricengine#Topology.add`
- `internal/fabricengine#Topology.rollbackAdd`
- `internal/fabricengine#ErrPairNotFound`
- `internal/fabricengine#Topology.Remove`
- `internal/fabricengine#pairState`
- `internal/fabricengine#Topology.inspectPair`
- `internal/fabricengine#Topology.refuseWhenNothingRemains`
- `internal/fabricengine#Topology.Prune`
- `internal/fabricengine#removeStalePair`
- `internal/fabricengine#Topology.repairPairWiring`
- `internal/fabricengine#writeLaunchers`
- `internal/fabriccli#Command`
- `internal/fabriccli/doc.go`
- `internal/fabricengine/doc.go`
- `internal/fabricengine_test#TestRemove_ReturnsPairNotFoundWhenNothingRemains`
- `plan:internal/fabricengine_test#TestHubWorkspace_TracksTopologyVerbs`
- `internal/fabricengine_test#TestAddLocal`
- `internal/fabricengine_test#requireNothingCreated`
- `internal/fabricengine_test#TestAddRollback_AdoptedWeftBranchSurvives`
- `internal/fabricengine_test#TestAddRollback_LiveWeftFromOrigin`
- `internal/fabricengine_test#TestAddRollback_WarpBranchDeletedUnderEmptyPrefix`
- `internal/fabricengine_test#TestAddRollback_AdoptedWarpBranchLocalCopyDeleted`
- `internal/fabricengine_test#TestAdd_RunRecords`
- `internal/fabricengine_test#VerbFixture`
- `internal/fabricengine_test#pairPermittedRoots`
- `internal/fabricengine_test#slugPermittedRoots`
- `internal/fabricengine_test#addCase`
- `internal/fabricengine_test#removeCase`
- `internal/fabricengine_test#pruneCase`
- `internal/fabricengine_test#TestVerbCases_CleanState`
- `internal/fabricengine_test#TestRemove_UnreachableOriginFailsClosed`
- `internal/fabricengine_test#TestAddRollback_LeavesNoArchiveTag`
- `internal/fabricengine_test#TestRemoveRefusal_ProbeLeavesPairUntouched`
- `internal/fabricengine_test#TestRemove_WarpStatusProbeFailurePushesNoTag`
- `internal/fabricengine_test#assertPairIntact`
- `internal/hubforge#TestNewHub`
- `internal/hubforge#TestCopyHub`
- `internal/fabriccli_test#requireAdoptedFromOrigin`
- `internal/fabriccli_test#TestRunCLI_AdoptRemoteWeftScenario`
- `internal/fabriccli_test#TestRunCLI_HubMutationScenario`
- `internal/battencli#TestBattenIntegration_CreateRow_IncompletePairRemedyWorksVerbatimOnAPrefixedHub`

**Delete:**
- `internal/fabricengine#RemoveStepPortal`
- `internal/fabricengine#ReconcileActionPortalRestored`
- `internal/fabricengine#restorePortalAndLaunchers`
- `internal/fabricengine_test#TestReconcile_RestoresDeletedPortalAndLaunchers`
- `internal/fabricengine_test#TestPrune_ApplyRemovesPortalAndLaunchers`

**Create:**
- `plan:internal/fabricengine#ReconcileActionLaunchersRestored` -> `const ReconcileActionLaunchersRestored ReconcileAction = "launchers_restored"`
- `plan:internal/fabricengine#restoreLaunchers` -> `func restoreLaunchers(rec *Mutations, warpLayout *lyxcwd.Location, slug string, pr *ReconcilePairResult) bool`
- `plan:internal/fabricengine_test#TestReconcile_RestoresDeletedLaunchers` -> `func TestReconcile_RestoresDeletedLaunchers(t *testing.T)`
- `plan:internal/fabricengine_test#TestPrune_ApplyRemovesLaunchers` -> `func TestPrune_ApplyRemovesLaunchers(t *testing.T)`

**Uses:**
- `internal/fabricengine#LauncherDir`
- `internal/fabricengine#removeLaunchers`
- `internal/fabricengine#localMarkOfCurrentBranch`
- `internal/hubforge#Hub.PairLauncherDir`
- `internal/hubforge#AddPairWith`
- `internal/hubforge#CopyHub`

**Intent:**
Stop every fabric verb from creating, restoring or removing a portal link (D6), keep the launcher restore, and report it as `launchers_restored`.

Verbs.
- `Topology.add` drops its `createPortal` step; the `!local` guard now wraps `writeLaunchers` alone, and the comment saying no hub workspace lists a local pair goes (D2): a local pair gets no launchers because it is local, not because the workspace skips it.
  `Add`'s and `AddLocal`'s docs stop naming a portal.
- `rollbackAdd` drops its portal removal step and its doc's mention of `removePortal`.
- `Remove` drops its portal removal and the `RemoveStepPortal` step; `pairState` loses its `portal` field, `inspectPair` stops probing the portal link, and `refuseWhenNothingRemains` no longer counts a portal as a pair remnant, so a pair whose only remnant is a portal link now returns `ErrPairNotFound` (`ErrPairNotFound`'s doc and `remove.go`'s file comment follow).
  `RemoveStepPortal` is deleted from the closed step set; no test or consumer names its value.
- `removeStalePair` drops its portal teardown; `Prune`'s doc names only the launcher directory.
- `repairPairWiring` calls `restoreLaunchers` in place of `restorePortalAndLaunchers`, and sets `ReconcileActionLaunchersRestored` where it set `ReconcileActionPortalRestored`: an otherwise healthy pair whose launcher directory was restored reports `launchers_restored`, so a consumer keying off the action still sees that convergence altered the pair.
- `restoreLaunchers` keeps the old function's prime skip and its detail-only failure handling, and restores only the launcher directory.
  It also skips a pair whose warp branch carries the local-only mark, read through `localMarkOfCurrentBranch(warpLayout.WorktreePath())`, since `AddLocal` creates no launcher directory; a mark read error is a Detail note and restores nothing, a fail-loud guard with no reaching test, since the same unreadable repository fails Reconcile's own `readBranch` of that worktree first and reports the pair `unmanaged_reported` before `repairPairWiring` runs.
  Bound: the new skip can only leave a launcher directory absent for a local pair, which never had one.
- `writeLaunchers`' doc names `restoreLaunchers` as its repair caller.
- `internal/fabricengine/doc.go`'s Remove-archive paragraph and `AddLocal` paragraph stop naming a portal.
- `fabriccli`'s help text (the `add` long text and `--local` flag, `remove`, `reconcile`, `prune`) and `internal/fabriccli/doc.go` stop naming portals, and `reconcile`'s text names `launchers_restored` (D8).

Tests.
Every test that planted a file at a portal path to make `Add` fail after both worktrees exist now plants it at `LauncherDir`/`Hub.PairLauncherDir` instead, which fails `writeLaunchers`' `root.MkdirAll` at the same step: `TestAddRollback_AdoptedWeftBranchSurvives`, `TestAddRollback_LiveWeftFromOrigin`, `TestAddRollback_WarpBranchDeletedUnderEmptyPrefix`, `TestAddRollback_AdoptedWarpBranchLocalCopyDeleted`, `TestAdd_RunRecords`'s rollback subtest and `TestAddRollback_LeavesNoArchiveTag`.
`removeLaunchers` on rollback meets a regular file where it expects a directory and fails without removing it, so each test's planted blocker survives as before.
The live-state matrix renames `VerbFixture.PortalBlocker` to `LauncherBlocker`, `addCase` plants it at `Hub.PairLauncherDir` (its error still names the slug through the launcher path), `TestVerbCases_CleanState` checks the renamed field, and `pairPermittedRoots`, `slugPermittedRoots`, `removeCase` and `pruneCase` drop the portal root and portal assertions (the launcher root and its anchor parent stay).
Tests that listed the portal among a pair's parts drop it: `TestAddLocal` (it still asserts no launcher dir), `requireNothingCreated`, `TestRemove_UnreachableOriginFailsClosed`, `TestRemoveRefusal_ProbeLeavesPairUntouched`, `TestRemove_WarpStatusProbeFailurePushesNoTag`, `assertPairIntact`, `requireAdoptedFromOrigin`, `TestRunCLI_AdoptRemoteWeftScenario`, `TestNewHub` (its subtest becomes `AddPairWiresLauncher`) and `TestCopyHub`.
`TestBattenIntegration_CreateRow_IncompletePairRemedyWorksVerbatimOnAPrefixedHub` stops planting a portal link as a stand-in for `Add`'s later steps, since `Add` creates none.
`TestRunCLI_HubMutationScenario`'s `Remove_RefusesDriftedPortalJunctionWithRefusalObject` subtest, the repo's one positive assertion that a `refusal` object reaches an envelope, needs a new gate refusal: it moves the pair's real launcher directory, scripts included, to a directory outside the hub and plants a symlink to it at the launcher path, so `removeLaunchers`' first script removal finds an existing target behind an escaping parent (`checkPathRequest` skips an absent target, so the scripts must be there) and `remove --force` exits 1 with a `refusal` object whose `check` is `containment`; the subtest is renamed to match, and its four-key and flattened-error assertions are unchanged.
`TestReconcile_RestoresDeletedLaunchers` replaces `TestReconcile_RestoresDeletedPortalAndLaunchers`: an ordinary pair whose launcher dir was deleted reports `launchers_restored` with the dir back, and a pair from `AddLocal` comes out of the same `Reconcile` with no launcher dir and an action other than `launchers_restored`.
`TestRemove_ReturnsPairNotFoundWhenNothingRemains` gains a row: with only a portal link left at `<hub>/_portals/<slug>` (planted by hand), `Remove` wraps `ErrPairNotFound` and leaves the link in place.
`TestHubWorkspace_TracksTopologyVerbs` asserts `<hub>/_portals` is absent after its `Add` and `AddLocal` steps, so no verb creates anything under `_portals`.
`TestPrune_ApplyRemovesLaunchers` replaces `TestPrune_ApplyRemovesPortalAndLaunchers` with the same hand-deleted-warp scenario asserting only the launcher dir is gone.

**ImpactSummary:** portal creation, restore and teardown leave Add, rollbackAdd, Remove, Prune and Reconcile; the reconcile action value changes from portal_restored to launchers_restored, and every test naming a portal path moves to the launcher path or drops it.

**Commit:** 3: fabric verbs stop creating, restoring and removing portal links

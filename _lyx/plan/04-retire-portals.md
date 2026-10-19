# Card 4 — retire-portals

**Edit:**
- `internal/fabricengine#Topology.add`
- `internal/fabricengine#Topology.AddLocal`
- `internal/fabricengine#Topology.rollbackAdd`
- `internal/fabricengine#Topology.Remove`
- `internal/fabricengine#pairState`
- `internal/fabricengine#Topology.inspectPair`
- `internal/fabricengine#Topology.refuseWhenNothingRemains`
- `internal/fabricengine#ErrPairNotFound`
- `internal/fabricengine#Topology.Prune`
- `internal/fabricengine#removeStalePair`
- `internal/fabricengine#applyStalePairOwnership`
- `internal/fabricengine#portalsDirName`
- `internal/fabricengine#HubReservedNames`
- `internal/fabricengine#refuseUncontainedPath`
- `internal/fabricengine#containmentFailure`
- `internal/fabricengine#pruneEmptyAncestors`
- `internal/fabricengine#writeLaunchers`
- `internal/fabricengine#hubRel`
- `internal/fabricengine#removeLaunchers`
- `internal/fabricengine#checkPathRequest`
- `internal/fabricengine#resolvePathOwnership`
- `internal/fabricengine#TestGate_ZeroValueDeclarationsAreRefusals`
- `internal/fabricengine#wantMenuLauncherName`
- `internal/fabricengine#TestRemoveLaunchers_PreservesForeignContent`
- `internal/fabricengine#TestRemoveLaunchers_DirRemovalIsContained`
- `internal/fabricengine#TestRemoveLaunchers_EmptyDirRemovedAndRecorded`
- `internal/fabricengine_test#TestCorrespondenceIndex`
- `internal/fabricengine_test#TestRemove_DoesNotDeleteOutsideHubThroughLauncherSymlink`
- `internal/fabricengine_test#TestRemove_ReturnsPairNotFoundWhenNothingRemains`
- `plan:internal/fabricengine_test#TestVerbsRegenerateHubWorkspace`
- `internal/fslink#RawTarget`
- `internal/fabricengine_test#TestAddRollback_AdoptedWeftBranchSurvives`
- `internal/fabricengine_test#TestAddRollback_LiveWeftFromOrigin`
- `internal/fabricengine_test#TestAddRollback_WarpBranchDeletedUnderEmptyPrefix`
- `internal/fabricengine_test#TestAddRollback_AdoptedWarpBranchLocalCopyDeleted`
- `internal/fabricengine_test#TestAddRollback_LeavesNoArchiveTag`
- `internal/fabricengine_test#TestAdd_RunRecords`
- `internal/fabricengine_test#requireNothingCreated`
- `internal/fabricengine_test#TestAddLocal`
- `internal/fabricengine_test#TestRemove_WarpStatusProbeFailurePushesNoTag`
- `internal/fabricengine_test#assertPairIntact`
- `internal/fabricengine_test#TestRemoveRefusal_ProbeLeavesPairUntouched`
- `internal/fabricengine_test#TestRemove_UnreachableOriginFailsClosed`
- `internal/fabricengine_test#VerbFixture`
- `internal/fabricengine_test#pairPermittedRoots`
- `internal/fabricengine_test#slugPermittedRoots`
- `internal/fabricengine_test#addCase`
- `internal/fabricengine_test#removeCase`
- `internal/fabricengine_test#pruneCase`
- `internal/fabricengine_test#TestVerbCases_CleanState`
- `internal/fabricengine/doc.go`
- `internal/hubforge#registerTeardown`
- `internal/hubforge#TestNewHub`
- `internal/hubforge#TestCopyHub`
- `internal/fabriccli#Command`
- `internal/fabriccli/doc.go`
- `internal/fabriccli_test#TestRunCLI_HubMutationScenario`
- `internal/fabriccli_test#requireAdoptedFromOrigin`
- `internal/fabriccli_test#TestRunCLI_AdoptRemoteWeftScenario`
- `internal/battencli#incompletePairRemedy`
- `internal/battencli#battenCLI.wire`
- `internal/battencli#TestBattenIntegration_CreateRow_IncompletePairRemedyWorksVerbatimOnAPrefixedHub`
- `cmd/lyx#uncontainedWriteAllowlist`
- `cmd/lyx#TestNoUncontainedWrite_FabricengineProductionSource`

**Delete:**
- `internal/fabricengine#createPortal`
- `internal/fabricengine#removePortal`
- `internal/fabricengine#ensureContainedLinkParent`
- `internal/fabricengine#PortalsDir`
- `internal/fabricengine#PortalLink`
- `internal/fabricengine#portalAnchorDir`
- `internal/fabricengine#portalTarget`
- `internal/fabricengine#RemoveStepPortal`
- `internal/fabricengine#TestRemovePortal_LeavesAnchorDirectory`
- `internal/fabricengine_test#TestAdd_DoesNotCreatePortalOutsideHubThroughContainerSymlink`
- `internal/fabricengine#TestPortalsDirAndLaunchersDir`
- `internal/fabricengine#TestMirroredPortalLauncherMethods`
- `internal/fabricengine#TestRemoveLaunchersAndPortal_ContainmentRefusalSurvivesSurfaceRefusal`
- `internal/fabricengine_test#TestPrune_ApplyRemovesPortalAndLaunchers`
- `internal/hubforge#Hub.PairPortalLink`

**Create:**
- `plan:internal/fabricengine#TestLaunchersDir` -> `func TestLaunchersDir(t *testing.T)`
- `plan:internal/fabricengine#TestMirroredLauncherMethods` -> `func TestMirroredLauncherMethods(t *testing.T)`
- `plan:internal/fabricengine#TestRemoveLaunchers_ContainmentRefusalSurvivesSurfaceRefusal` -> `func TestRemoveLaunchers_ContainmentRefusalSurvivesSurfaceRefusal(t *testing.T)`
- `plan:internal/fabricengine_test#TestPrune_ApplyRemovesLaunchers` -> `func TestPrune_ApplyRemovesLaunchers(t *testing.T)`

**Prosa:**
- `README.md`
- `docs/overview.md`
- `docs/sandbox-hub.md`
- `docs/shared-libs/lyxcwd.md`
- `pattern/PATTERN-hub-containment.md`
- `pattern/PATTERN-hub-suffix.md`
- `tools/sandbox/SANDBOX-FABRIC-SUITE.md`

**Uses:**
- `internal/fabricengine#LauncherDir`
- `internal/fabricengine#launchersDir`
- `internal/fabricengine#RecordsWorktreePath`
- `internal/fabricengine#WorktreePath`
- `internal/hubforge#Hub.PairLauncherDir`
- `internal/hubforge#CopyHub`
- `internal/hubforge#AddPair`
- `internal/hubforge#AddPairWith`
- `internal/fslink#CreateDirLink`

**Intent:**
No verb creates or removes a portal link any more (D6).
After card 3 nothing restores one either, so this card retires the rest of the `_portals` surface.

Production:
- `Topology.add` creates launchers only, for an ordinary pair; a local pair still gets none.
  The comment saying no hub workspace lists a local pair goes, since the workspace lists every pair (D2).
  `AddLocal`'s doc says no launcher is created.
- `Topology.rollbackAdd` drops its portal step.
- `Topology.Remove` drops its portal teardown and the `portal` step.
  `pairState`, `Topology.inspectPair` and `Topology.refuseWhenNothingRemains` drop the portal presence probe, and `ErrPairNotFound`'s doc no longer names a portal.
  A pair whose only remnant is a portal link is now reported as not found, and the D7 sweep (card 5) removes that link.
- `removeStalePair` drops its portal teardown, and `Topology.Prune`'s doc names the launcher directory only.
- The portal helpers go: `createPortal`, `removePortal`, `ensureContainedLinkParent`, `PortalsDir`, `PortalLink`, `portalAnchorDir` and `portalTarget`.
  `portalsDirName` stays in `portals.go`, the token's registered owner file, because `HubReservedNames` still reserves `_portals` and card 5's sweep needs the name.
  Its comment, and the `portals.go` header, say `_portals` is retired, still reserved and swept by `lyx fabric reconcile`.
- Comments that name the portal create or teardown path are reworded to the launcher surface alone, or to `_portals` as retired: `HubReservedNames`, `refuseUncontainedPath` and the `ancestors.go` header, `containmentFailure`, `pruneEmptyAncestors`, `writeLaunchers`, `hubRel`, `removeLaunchers` and `checkPathRequest`.
  The same goes for:
  - the containment section of `internal/fabricengine/doc.go`, and the `AddLocal` and archive lines of that file;
  - the `remove.go` header and `resolvePathOwnership`'s one-hop comment;
  - `applyStalePairOwnership`'s doc, which says `removeStalePair` still has portal, launcher and worktree-registration debris to clear, and names launcher and worktree-registration debris only;
  - `fslink`'s `RawTarget` example in both platform files;
  - the comment in `TestGate_ZeroValueDeclarationsAreRefusals`, and the file headers above `TestCorrespondenceIndex` and `TestRemove_DoesNotDeleteOutsideHubThroughLauncherSymlink`;
  - the `portallauncher_test.go` header, which names the launcher accessors and the `_launchers` layout only, and the doc comment of `TestRemoveLaunchers_PreservesForeignContent`, which drops the portal half of the teardown;
  - `hubforge`'s `registerTeardown`;
  - `battencli`'s `incompletePairRemedy` and `battenCLI.wire`.
- `cmd/lyx`'s uncontained-write guard: its header comment, the `uncontainedWriteAllowlist` comment and the failure message of `TestNoUncontainedWrite_FabricengineProductionSource` name `launchers.go`'s `writeLaunchers` and `hubworkspace_write.go`'s `writeHubWorkspace` as the rooted writers, in place of `portals.go`.
- `internal/fabriccli#Command`'s help text for `add --local`, the `--local` flag, `remove` and `prune`, and `internal/fabriccli/doc.go`, drop the portal.
- `hubforge`'s `Hub.PairPortalLink` goes.

Tests, each dropping the portal path from its assertions:
- The Add rollback tests that planted a blocker file at the portal path to fail Add after its weft branch exists plant a directory at the pair's `ide` launcher script path, `<LauncherDir(slug)>/ide<ext>`, instead: `TestAddRollback_AdoptedWeftBranchSurvives`, `TestAddRollback_LiveWeftFromOrigin`, `TestAddRollback_WarpBranchDeletedUnderEmptyPrefix`, `TestAddRollback_AdoptedWarpBranchLocalCopyDeleted`, `TestAddRollback_LeavesNoArchiveTag` and `TestAdd_RunRecords`.
  `writeLaunchers` then fails writing the script, and `rollbackAdd`'s `removeLaunchers` removes the planted directory and the launcher directory, so each test keeps asserting a full rollback.
- `addCase` in the cross-product test plants the same launcher-script blocker, and `VerbFixture.PortalBlocker` becomes `LauncherBlocker`, which `TestVerbCases_CleanState` checks is a directory.
  The rollback now removes the planted blocker and its launcher directory, so `addCase`'s default expectation gains `PermittedRoots: slugPermittedRoots("verb-add-owner")`.
- `pairPermittedRoots`, `slugPermittedRoots`, `removeCase` and `pruneCase` drop the portal path and its anchor parent.
- `requireNothingCreated`, `TestAddLocal`, `TestRemove_WarpStatusProbeFailurePushesNoTag`, `assertPairIntact`, `TestRemoveRefusal_ProbeLeavesPairUntouched` and `TestRemove_UnreachableOriginFailsClosed` drop the portal path from their lists.
  `TestAddLocal` also asserts no `_portals` entry exists for the local pair.
- Four tests whose names would keep naming the portal are replaced under launcher-only names:
  - `TestPrune_ApplyRemovesLaunchers` replaces `TestPrune_ApplyRemovesPortalAndLaunchers` and asserts the launcher directory's removal alone.
  - `TestLaunchersDir` and `TestMirroredLauncherMethods` replace `TestPortalsDirAndLaunchersDir` and `TestMirroredPortalLauncherMethods`, keeping their launcher assertions with `//testtiming:keep` reasons that name launchers only.
  - `TestRemoveLaunchers_ContainmentRefusalSurvivesSurfaceRefusal` replaces `TestRemoveLaunchersAndPortal_ContainmentRefusalSurvivesSurfaceRefusal` without its portal row.
- `TestRemove_ReturnsPairNotFoundWhenNothingRemains` gains a row: after a pair is finished, a link planted by hand with `fslink.CreateDirLink` at `_portals/<AnchorRel>/<slug>` is all that remains, and `Remove` wraps `ErrPairNotFound`.
- `TestVerbsRegenerateHubWorkspace` asserts after each verb step that no `_portals` entry exists.
- `TestRemovePortal_LeavesAnchorDirectory` and `TestAdd_DoesNotCreatePortalOutsideHubThroughContainerSymlink`, with its file, go.
- `TestNewHub` and `TestCopyHub` drop the portal-link assertions and assert no `_portals` directory exists after a pair is added.
- `TestRunCLI_AdoptRemoteWeftScenario` and `requireAdoptedFromOrigin` drop the portal path.
- `TestRunCLI_HubMutationScenario`'s `Remove_RefusesDriftedPortalJunctionWithRefusalObject` row keeps the repo's positive assertion that a `refusal` object reaches the remove envelope.
  It drives the refusal through the launcher surface instead: the pair's launcher directory, scripts included, is moved to a directory outside the hub and an `fslink.CreateDirLink` link to it is left in its place, as the replaced row's drifted link was.
  `removeLaunchers`' first script removal then reaches `checkPathRequest`, whose containment check resolves the script's parent through that link to outside `_launchers` and refuses with `CheckContainment`.
  `refuseUncontainedPath` on the launcher directory itself does not refuse, because the leaf link is not resolved.
  The row asserts the four refusal keys, the flattened error and `check` equal to `containment`, and is renamed to name the launcher.
- `TestBattenIntegration_CreateRow_IncompletePairRemedyWorksVerbatimOnAPrefixedHub` stops planting a portal link and keeps every other step.

Docs in this card:
- `README.md`, `docs/overview.md`, `docs/sandbox-hub.md`, `docs/shared-libs/lyxcwd.md` and `pattern/PATTERN-hub-suffix.md` drop the live portal from their hub trees and path lists, and name `_portals` as retired where they list reserved names.
  This includes `overview.md`'s unimplemented "Portals" status line, and its batten note on a half-removed pair, which names launcher entries only.
- `pattern/PATTERN-hub-containment.md` reads: `_board` and `_launchers` are reachable from the hub only; `_launchers` links point hub-inward only; `_portals` is retired and stays reserved.
- `SANDBOX-FABRIC-SUITE.md` drops `_portals` from the hub listing, adds `<RepoName>.code-workspace` to it, drops F17's portal steps, and its local-pair step checks only `_launchers/<slug>`.

**ImpactSummary:** Removes the portal create, remove and probe paths from Add, AddLocal, Remove and Prune, and deletes seven portal helpers and `Hub.PairPortalLink`; every caller is a test, help text or comment listed here.

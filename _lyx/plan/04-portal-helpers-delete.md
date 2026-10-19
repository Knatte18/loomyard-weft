# Card 4 — portal-helpers-delete

**Delete:**
- `internal/fabricengine#createPortal`
- `internal/fabricengine#ensureContainedLinkParent`
- `internal/fabricengine#removePortal`
- `internal/fabricengine#PortalLink`
- `internal/fabricengine#PortalsDir`
- `internal/fabricengine#portalAnchorDir`
- `internal/fabricengine#portalTarget`
- `internal/hubforge#Hub.PairPortalLink`
- `internal/fabricengine#TestRemovePortal_LeavesAnchorDirectory`
- `internal/fabricengine/portals_containment_integration_test.go`
- `internal/fabricengine#TestPortalsDirAndLaunchersDir`
- `internal/fabricengine#TestMirroredPortalLauncherMethods`
- `internal/fabricengine#TestRemoveLaunchersAndPortal_ContainmentRefusalSurvivesSurfaceRefusal`

**Create:**
- `plan:internal/fabricengine#TestLaunchersDir` -> `func TestLaunchersDir(t *testing.T)`
- `plan:internal/fabricengine#TestMirroredLauncherMethods` -> `func TestMirroredLauncherMethods(t *testing.T)`
- `plan:internal/fabricengine#TestRemoveLaunchers_ContainmentRefusalSurvivesSurfaceRefusal` -> `func TestRemoveLaunchers_ContainmentRefusalSurvivesSurfaceRefusal(t *testing.T)`

**Edit:**
- `internal/fabricengine#portalsDirName`
- `internal/fabricengine#TestGate_ZeroValueDeclarationsAreRefusals`
- `internal/fabricengine_test#TestWarpLayoutFor_FastPathMatchesResolveWorktree`
- `internal/fabricengine_test#TestCorrespondenceIndex`
- `internal/lyxcwd#TestEnforcement_GeometryLiterals`
- `internal/fabricengine#refuseUncontainedPath`
- `internal/fabricengine#containmentFailure`
- `internal/fabricengine#pruneEmptyAncestors`
- `internal/fabricengine#checkPathRequest`
- `internal/fabricengine/doc.go`
- `cmd/lyx#uncontainedWriteAllowlist`
- `cmd/lyx#TestNoUncontainedWrite_FabricengineProductionSource`

**Intent:**
Delete the portal surface Card 3 left without a production caller (D6), keeping only what the reserved name set and Card 5's sweep need.

`portals.go` keeps `portalsDirName` alone, its doc rewritten: `_portals` is a retired hub container, still reserved (`HubReservedNames`, unchanged) so no slug can claim it, and swept by `lyx fabric reconcile`; the file comment says the same.
`createPortal`, `ensureContainedLinkParent`, `removePortal`, `PortalLink`, `PortalsDir`, `portalAnchorDir` and `portalTarget` are deleted, and so is `hubforge`'s `Hub.PairPortalLink`; Card 3 removed every caller outside these and the tests below.

Tests.
`TestRemovePortal_LeavesAnchorDirectory` and `portals_containment_integration_test.go` (`TestAdd_DoesNotCreatePortalOutsideHubThroughContainerSymlink`) are deleted with the behavior they pinned: no verb creates or removes a portal any more.
Three tests whose names advertise portal coverage are replaced under names that say what they still pin (the hub has no `rename_tools`, so a symbol `Rename` is refused; each is a delete and a create of the same body less its portal part, so no behavior is covered twice):
`TestLaunchersDir` is `TestPortalsDirAndLaunchersDir` keeping its `launchersDir` and `LauncherDir` assertions and dropping `PortalsDir` and `portalTarget`;
`TestMirroredLauncherMethods` is `TestMirroredPortalLauncherMethods` without its `PortalLink` subtest;
`TestRemoveLaunchers_ContainmentRefusalSurvivesSurfaceRefusal` is `TestRemoveLaunchersAndPortal_ContainmentRefusalSurvivesSurfaceRefusal` keeping the launcher row and dropping its portal row and its `_portals` symlink plant.
Their `//testtiming:keep` reasons and doc comments, and `portallauncher_test.go`'s file comment, name only what they assert.
Test comments that name deleted portal code as live are reworded: `TestGate_ZeroValueDeclarationsAreRefusals` drops `removePortal` from its idempotent-teardown list; the file comments of `warplayout_test.go` (held by `TestWarpLayoutFor_FastPathMatchesResolveWorktree`) and `index_integration_test.go` (held by `TestCorrespondenceIndex`) stop naming `_portals` or portal wiring as part of a hub; `livestate_doc_test.go`'s header comment, which holds no member to name, drops the portal from remove.go's teardown list; `TestEnforcement_GeometryLiterals`' comment says `_portals` is a retired, still-reserved name whose constant `internal/fabricengine` keeps, not live plumbing.

Comments that name the deleted helpers as live code are reworded: `refuseUncontainedPath`'s and `containmentFailure`'s docs name `removeLaunchers` as the remaining teardown caller and launchers.go as the one non-gate caller, `pruneEmptyAncestors`' doc, and `ancestors.go`'s file comment above it, name `_launchers` as the container it walks and drop portals, `checkPathRequest`'s doc drops `removePortal` from its idempotent-teardown list, and `internal/fabricengine/doc.go`'s write-containment and teardown sections say the portal writer and remover were retired, `writeLaunchers` being the one hub-container writer left.
`cmd/lyx`'s `uncontainedWriteAllowlist` doc and `TestNoUncontainedWrite_FabricengineProductionSource`'s failure message stop pointing at `portals.go`'s `ensureContainedLinkParent` and point at `launchers.go`'s `writeLaunchers` alone.

**ImpactSummary:** removes seven fabricengine symbols and one hubforge accessor whose last callers Card 3 removed; the rest is comment and test pruning inside fabricengine and one cmd/lyx guard message.

**Commit:** 4: delete the retired portal helpers

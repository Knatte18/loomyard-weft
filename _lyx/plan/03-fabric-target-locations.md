# Card 3 — fabricengine builds another pair's paths at that pair's anchor

**Edit:**
- `internal/fabricengine#ReadOriginFor`
- `internal/fabricengine#WriteOrigin`
- `internal/fabricengine#MarkBeam`
- `internal/fabricengine#Topology.removePair`
- `internal/fabricengine#Topology.inspectPair`
- `internal/fabricengine#Topology.RemoveRefusal`
- `internal/fabricengine#commitPendingRecords`
- `internal/fabricengine#ResetRecordsToParent`
- `internal/fabricengine#hubWorkspaceFolders`
- `internal/fabricengine#regenerateHubWorkspace`
- `internal/fabricengine#sweepRetiredHubArtifacts`
- `internal/fabricengine#pairHasLocalPart`
- `internal/hubforge#Hub.PairLauncherDir`
- `internal/fabricengine_test#TestRegenerateHubWorkspace`
- `internal/fabricengine_test#TestRemove_TearsDownNestedJunction`
- `internal/fabricengine_test#TestReconcile_SweepsRetiredHubArtifacts`

**Create:**
- `internal/hubforge#ReanchorPair`

**Uses:**
- `internal/lyxcwd#NamedWorktree`
- `internal/fabricengine#OriginRecordPathFor`
- `internal/fabricengine#LauncherDir`
- `internal/fabricengine#removeLaunchers`
- `internal/fabricengine#refuseDirtyWeftWorktree`
- `internal/fabricengine#replaceRecordsContent`
- `internal/fabricengine#missingOriginRecordError`
- `internal/fabricengine#retiredPortalTarget`
- `internal/fabricengine#retiredPortalChain`
- `internal/fabricengine#retiredHubWorkspacePath`

**Intent:**
D3 inside `fabricengine`, following the Shared Decisions rule for `(l, slug)` builders.
Each function below that acts on a worktree other than the one its `l` names gets that worktree's Location once, `pair, err := lyxcwd.NamedWorktree(l.HubPath, slug)`, and hands `pair` as the `l` of every anchored builder and every `anchorRel` argument it passes for that pair:
`ReadOriginFor`, `WriteOrigin` and `MarkBeam` (the origin record path, its commit's anchor and `missingOriginRecordError`'s path);
`Topology.removePair`, `Topology.inspectPair` and `Topology.RemoveRefusal` (launcher dir, junction teardown, `refuseDirtyWeftWorktree`'s anchor);
`commitPendingRecords` (the present-name probe and the commit's anchor);
`ResetRecordsToParent` (its `WriteOrigin`, `replaceRecordsContent`'s run-records root and the commit's anchor);
`pairHasLocalPart` (the launcher dir).
A pair whose records worktree is gone resolves to D2's fallback; such a caller uses the fallback only for the hub-level launcher directory, and no anchored path inside the pair (D3).

`regenerateHubWorkspace` reads the prime's VS Code settings, and `hubWorkspaceFolders` places the prime's folder, at the prime's own anchor, from `NamedWorktree(l.HubPath, primeName)`, so a regeneration run from a pair at another anchor writes the prime's folder correctly; each pair's folder already comes from its own `ResolveWorktree`.
`sweepRetiredHubArtifacts` keys the retired-portal sweep and the retired hub workspace path on the prime's anchor, the same way, whatever worktree reconcile runs from; portals left under an old prime anchor are not swept (D3).

`hubforge.ReanchorPair` is the test-only stand-in for an anchor change, until card 15 builds one, for any worktree of the hub, the prime included (`slug` its worktree name): in the worktree's records worktree it moves each wired name's tracked content from its current anchor to `anchor` with `git mv`, writes `anchor` into the root `.lyx-anchor`, and commits both in one commit; in its code worktree it commits a tracked placeholder `<anchor>/.keep` when git does not already track `anchor` there, so the anchor is a directory git tracks as `validateAnchorTarget` and checkout's check require; then it unwires the worktree's junctions and wires them at `anchor` through `fabricengine`'s wiring.
Cards 3 to 14 use it to build a hub with two anchors.

Tests:
`TestRegenerateHubWorkspace` gains a row with a beam pair re-anchored by `ReanchorPair` at `sub`: its hub-workspace folder and its beam workspace file name `<slug>/sub`, the prime's folder stays at the prime's anchor, and a regeneration run from that pair's Location writes the same file.
`TestRemove_TearsDownNestedJunction` gains a subtest: a pair re-anchored at `sub` in a root-anchored hub is removed from the prime, its junctions at `sub` and its launcher directory `_launchers/sub/<slug>` gone, and nothing was looked for at the prime's anchor.
It gains a second subtest for D3's gone weft: a pair re-anchored at `sub` whose records worktree is then deleted by hand answers `ReadOriginFor` not found with no error, and its removal succeeds, removing the warp worktree with no step that builds a path under `<slug>/sub`.
`TestReconcile_SweepsRetiredHubArtifacts`, which already runs each case from the prime and from a pair, gains a third `from` value: a pair re-anchored at `sub`, which sweeps the portals keyed on the prime's anchor.

**ImpactSummary:** no signature changes; ten `fabricengine` functions read the target pair's own anchor, which equals the acting one in every one-anchor hub, plus one hubforge accessor and one new hubforge helper.

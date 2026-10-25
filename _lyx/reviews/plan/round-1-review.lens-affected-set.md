```yaml
lens: affected-set
outcome: reported
```

Lens: affected-set. For each output, key, string, file shape or behaviour a card changes, I listed what reads or asserts it and which cards cover it.

**Per change: the set found, and the cards covering it**

1. Card 1 re-signs `readRecordedAnchor(hub)` to `(hub, worktreeName)`.
   - Set: `lyxcwd.go:130` `resolveCore`, and `anchorread_test.go:55` (`TestReadRecordedAnchor_MarkerSemantics`).
   - Covered by card 1.
   - The gate message `ErrCwdOutsideAnchor` naming `AnchorFileName`, asserted by `webstercli/cli_integration_test.go:145` and `preflight_integration_test.go:560`, does not change.
2. Card 2 adds an error result to `pairLocation`.
   - Set: `beamcli/dobbywatch.go:34` (`dobbyWatchDirs`) and `beamcli/dobbydeps.go:342` (`dobbyRuns`).
   - Covered by card 2.
3. Card 2 replaces the production `Location` literals.
   - Set: `dobbydeps.go:232`, `clone.go:290`, `warplayout.go:33`, `teardown.go:336`. The map and slice literals in battencli, orchcli and tendercli are correctly excluded.
   - Covered by card 2.
4. Card 6 moves the marker off Bolt onto main-weft. Readers and seeders:
   - The anchor-path tests in `clone_adopt_test.go`, `warpbinding_clone_integration_test.go`, `remove_junctions_integration_test.go` (`seedAnchorMarker`, `TestRemove_TearsDownNestedJunction`), `status_pollution_integration_test.go`, `unwire_test.go`, `reconcile_empty_anchor_integration_test.go`, `creelcli/delivery_test.go`, `preflight_integration_test.go` (`TestHealthyHub`, `TestResolveMode`), `hubforge/hub_test.go` (`assertRealHub`) and `fabriccli/cli_test.go` (`TestRunCLI_CloneEndToEnd`) are covered by card 6.
   - Bolt-seeded clone tests that still work through card 6's Bolt fallback, so they need no change: `TestCloneHub_ExplicitRootSubpathRefusesRecordedSubpath`, `TestCloneHub_BackfillsBindingOnPreBindingHub`, `TestRunCLI_CloneOfAnUnmigratedWeftLeavesTheMarkerToTheWalk`.
   - `webstercli/sync_integration_test.go` `seedFabricAnchor` runs over a plain gitkit repo, so the Bolt fallback still holds and no change is needed.
   - Unverified: `commit_integration_test.go` `writeFabricAnchor` over `NewCommitFixtureForTest`. I could not tell whether that fixture is hubforge-built. If it is, the subtest at line 305 is masked by the prime's own marker.
   - `cmd/lyx/uncontainedwrite_test.go`'s `clone.go` allowlist entry: no card covers it (finding 1).
   - Docs and comments that still place `.lyx-anchor` on `weft:main`/Bolt: `fabricengine/doc.go` is covered by cards 6 and 8. `unwire.go:5`, `fabriccli/fabric.go:423` and `codebinding.go:2` are not covered (finding 2).
5. Card 8 deletes `refuseEmptyAnchorMarker`.
   - Set: `reconcile.go:168` and `reconcile_empty_anchor_integration_test.go`.
   - Covered by card 8.
6. Card 9 adds the push class `anchor_change`.
   - Set: no consumer of the `PushFailureClass` constants outside `fabricengine` (`failedpush.go:21-36` and `pushclassified.go` only).
   - Covered by card 9.
7. Card 11 adds the branch state `anchor_stopped`.
   - Set: the CLI switches are `fabriccli/remove.go:160`, `hibernatecli/reconcile.go`, `cleanupcli/reconcile.go` and `adoptcli/adopt.go`.
   - The `fabricengine` hits in `parenttips.go`, `bolt.go` and `reconcile.go` do not switch on these state constants.
   - Covered by card 11, together with battencli `judgeOrigin`.
8. Card 18 re-signs `create` and `createSteps`.
   - Set: `create.go:75`, `promote.go:97`, `cli.go:204`, and `create_test.go:222` and `:261`.
   - Covered by card 18.
9. Card 16 adds a `Geometry` field and a `StartDeps` field.
   - Set: the hand-built `dobbyengine.Geometry{}` in `beamcli/dobby_test.go:23-24` and `beamcli/cli.go:141-169` gate nothing when empty. `dobbyengine/seam_enforcement_test.go` pins only imports.
   - Covered by card 16.

**Findings**

1. **MEDIUM**, class `design`.
   - Location: card 6 (`commitAnchorMarker`), card 15 (the D7 move's marker write, the `.lyx` rename and the record write), card 8 (the marker write in the migration).
   - Summary: new raw filesystem writes in `internal/fabricengine` are subject to the per-file write allowlist `uncontainedWriteAllowlist` in `cmd/lyx/uncontainedwrite_test.go`, and no card names that set.
     - The `clone.go` entry's `Why` says "the `.lyx-anchor` marker [is] written into the `_board` worktree". After card 6 that is false: clone writes the marker into the prime's weft worktree, through `commitAnchorMarker`.
     - None of these cards says which file `commitAnchorMarker` or the move's writes live in. Placed outside an allowlisted file, they fail the tier0 scan. Placed inside one, the stale `Why` stands.
   - Missed members: the `uncontainedwrite_test.go` allowlist entries for each new writer's file, and the reworded `clone.go` `Why`.
   - Evidence: `cmd/lyx/uncontainedwrite_test.go:3-5`, `:70-72`; `clone.go:372-391`.
2. **LOW**, class `consistency`.
   - Location: cards 6 and 8, docs.
   - Summary: the doc comments and help text saying `.lyx-anchor` is a repo-wide `weft:main` (Bolt) record are not updated by any card. Card 6 covers only `fabricengine/doc.go`, the probe field docs and the guard message.
   - Missed members:
     - `internal/fabricengine/unwire.go:5` ("never touches the repo-wide `weft:main` records (`.lyx-anchor`, …")
     - `internal/fabriccli/fabric.go:423` ("(.lyx-anchor, the .lyx-warp binding, and fabric.yaml)")
     - `internal/fabricengine/codebinding.go:2` ("recorded once on weft:main beside .lyx-anchor")
     - the clone stale-guard comment at `clone.go:33`
   - Evidence: grep output for `AnchorFileName`/`.lyx-anchor` in production code.

Tool calls:

1. Grep for callers of `readRecordedAnchor`, `refuseEmptyAnchorMarker`, `pairLocation`, `create` and `createSteps`, plus the test files touching the marker: the manifest held the definitions, not the call sites.
2. Per-file marker uses in those test files, with their enclosing functions: to see which tests seed or assert the Bolt marker beyond card 6's list.
3. `writeFabricAnchor`, `seedFabricAnchor`, `TestRunCLI_CloneOfAnUnmigratedWeftLeavesTheMarkerToTheWalk` and `uncontainedwrite_test.go`: helpers and the allowlist the manifest lacked.
4. Users of `BranchDiverged` and `PushFailureClass`, and the fixture kinds of the seeders: consumers of the new state and class.
5. The `BranchState` switches in `parenttips`, `reconcile`, `bolt` and `remove`, plus fixture kinds: to confirm or rule out switches card 11 misses.
6. `dobbyengine` seam enforcement and the hand-built `Geometry`/`StartDeps` literals: consumers of card 16's new fields.

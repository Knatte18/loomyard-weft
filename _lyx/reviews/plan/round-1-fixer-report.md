# Fixer report — plan, round 1

## Changes

- **F1** (fixed as proposed): removed the `internal/planglyph#deleteOrderFixture` Edit bullet from card 1.
  `deleteorder_test.go:16` holds the fixture map, and no new row changes it, since every new row's files go in `extraFiles`.
- **F2** (fixed as proposed): card 4 re-signs `internal/planglyph#CanonicalizeHandles` without `planDir`, lists its seven further test callers in `handle_test.go` under Edit, and its Intent says the parameter is dropped and every caller stops passing it.
  The premise holds: `handle.go:345` is the function's only use of `planDir`.
  Follow-through in the same finding: card 4's ImpactSummary no longer says the callers see no change, and now says they drop the argument with no behavior change for a disk plan.
- **F3** (fixed as proposed): card 1's "never walking the tree" now limits itself to candidate files and notes that `coverageSubjects`' go.mod walk still runs once.
  The premise holds, since `coverageSubjects` calls `planparser.NestedModules`, which calls `goModDirsOnDisk`.
- **F4** (fixed as proposed): card 1's `TestGlyphChain_CallerCoverageTyped` bullet says `countingTypesLoader` and its load method live in `glyphchain_integration_test.go` beside the test.

## Disputed

None.

## Deferred

None.

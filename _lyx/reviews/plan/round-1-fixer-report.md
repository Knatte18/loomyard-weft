# Fixer report — plan, round 1

## Changes

- **F1** (fixed as proposed): card 5's Edit list gains `internal/reedengine/render#TestDisplayMarshalsWithoutRetiredKeys`, `internal/reedengine#goodAttachStrands`, `internal/reedengine#TestAttachArgv_EveryOtherDegradedPathYieldsBareArgv` and `internal/reedengine#setupAttachGeometryFixture` after `TestStrandWindowTarget`.
  Its Tests paragraph says the four move their displays to `own-window`, and that `TestDisplayMarshalsWithoutRetiredKeys` round-trips an `own-window` display with `KeepViewer` set.
  The other retired-constant uses in the tree (`attach_test.go:614-615`, `attachgeometry_integration_test.go:319`) sit in tests card 2 deletes.
- **F2** (fixed as proposed): card 2's Intent says the `render/policy.go` file header is rewritten to describe only `ValidateDisplay`'s display check, dropping the placed-cell filtering, stack ordering, cycle repair, below-parent and top wording and the policy-versus-mechanics note.
- **F3** (fixed as proposed): card 8's Prosa list gains `internal/reedcli/smoke_test.go` and `internal/reedcli/smoke_lifecycle_test.go`, and its Intent says both headers drop the layout-survival-under-stacked-below-parent-adds scenario.
- **F4** (fixed as proposed): card 2's Delete list replaces the `TestPartitionByAnchor` and `TestBreakCyclesTerminatesAndKeepsEveryStrand` bullets with `internal/reedengine/render/policy_test.go`, and its Tests paragraph names that file among those that go whole.
- **F5** (fixed as proposed): card 5's Edit list drops `internal/reedengine#TestWouldFormCycle`.

## Disputed

None.

## Deferred

None.

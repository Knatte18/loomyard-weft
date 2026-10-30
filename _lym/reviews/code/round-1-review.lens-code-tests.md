```yaml
lens: code-tests
outcome: reported
```

**code-tests lens: for each behaviour the diff adds or changes, the test that pins it, or none.**

- **Preflight is the pawl recipe's entry, `PawlRouting().Entry == Preflight`.**
  Pinned by `TestPawlRouting`, `internal/loomrecipe/pawlrecipe_test.go:166`. The assertion is changed in the diff.
- **Preflight is built as the general producer type (`preflightshed.NewPreflight`).**
  Pinned by the `wantPawlRows` row in `TestNewPawl/BuildsTheRowsAndTheirEdges`, `internal/loomrecipe/pawlrecipe_test.go:40`. It compares the producer's `reflect.Type`.
- **Preflight hands to Beam-Up, with no `on_stuck` (`onStuck` is `""`).**
  Pinned by the same `wantPawlRow`, checked in `TestNewPawl` at `pawlrecipe_test.go:75` and in `TestPawlRouting` at `:175`.
- **The row order has Preflight first.**
  Pinned by `TestWire_BeamRecipe`, `internal/loomcli/wiring_beam_test.go:94`.
  `TestPawlRouting` also checks the order row by row against `wantPawlRows`.
- **Preflight is hold-free for a beam project (`holdFreeProducers`).**
  Pinned by two tests:
  - `TestRunHolds/BeamAtPreflight`, `internal/boardengine/runlock_test.go:366`.
  - `TestPawlRecipe_HoldLiftsOnlyBeforeParentAlign`, `internal/loomrecipe/pawlrecipe_test.go:198`. It walks `RowNames(pawl)`, so it covers Preflight through the recipe and would fail if the hold-free list lacked it.
- **A non-beam task at Preflight still holds.**
  Pinned by `TestRunHolds/TaskAtAHoldFreeProducerName`, `runlock_test.go:363`. It uses `Beam-Await`, not `Preflight`, but `Preflight` takes the same code path through `slices.Contains`, so the case is covered. I raise nothing.
- **`runTakesNoSlot` is true at Preflight (the slot rule, `internal/battencli/dobbyreads.go:180`).**
  None. `TestRunTakesNoSlot` (`internal/battencli/dobbyreading_test.go:119`) has rows for Beam-Await, Worktree-Create and Parent-Align, none for Preflight.
  The behaviour is derived from `RunHolds`, which is pinned, and the new row would duplicate the `Beam-Await` row's code path. I record it as LOW, a judgment call, not a blocking gap.
- **A pawl run's `Env.Cwd` is absolute, so the `Preflight` registry row builds for a pawl seed.**
  The registry row's own `requireAbsRoot` refusal is tested in `internal/shedrecipe`, but I did not verify that. `Env.Cwd` is set unconditionally at `internal/loomcli/wiring.go:556`, before the pawl branch.
  `TestWire_BeamRecipe` runs `wire` with `hubLocation(t, "pair", ".")` and then `recipeRouting`. I did not open that test's body to confirm it also builds the recipe, which is what would exercise `requireAbsRoot` for a pawl seed. Treat the pinning test as unconfirmed.
- **Test economy.**
  No added test duplicates an existing one. The diff adds one table row and edits assertions and names, and adds no new test function.

Findings:

1. LOW, `design` (coverage gap), `internal/battencli/dobbyreading_test.go:119`: `TestRunTakesNoSlot` has no `BeamProjectAtPreflight` row, so the dobby slot rule for the new hold-free row is not pinned directly. Evidence: the diff changes `dobbyreads.go:179` and `dobbyengine/doc.go:104` to name Preflight, and the table's only no-slot rows are `Beam-Await` and `Worktree-Create`.

Tool calls (one):
1. `grep -n "HoldLifts" … ; grep OnStuck …; sed -n 100,145p dobbyreading_test.go`. This read `TestPawlRecipe_HoldLiftsOnlyBeforeParentAlign`, the `OnStuck` assertions and `TestRunTakesNoSlot`, whose bodies were not in context.

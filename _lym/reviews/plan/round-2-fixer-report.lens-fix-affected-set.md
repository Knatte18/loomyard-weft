```yaml
lens: fix-affected-set
outcome: reported
```

I checked both findings' sets against the code. One gap: after the fix, no test confirms that battencli fills the new seams.

Unchecked findings: none. I re-ran the searches for F16 and F17.

**F16.** The set was every test that calls `envkit.NilSeams` against an `intentionallyNil` list. Those are loomcli's `wiring_seams_test.go`, plus `wiring_darn_test.go` and `wiring_beam_test.go` in loomcli, and battencli's `wire_seams_test.go`. Card 10 now adds `Teardown.Push` and `Teardown.Level` to no `intentionallyNil` list, so none of these tests can trip on them. That part is covered.

- **LOW — card 10, TeardownDeps bullet: "`NilSeams` then skips both, so loomcli's `intentionallyNil` stays as it is".**
  - Problem: once both paths are in `nilLegal`, battencli's `TestWire_EverySeamFilled` also skips them. Nothing then checks that `battenCLI.wire` fills `Push` and `Level`, which the card's wire bullet promises.
  - Evidence: `internal/battencli/wire_test.go:64-65` (`TestWire_BuildsLazilyForANonexistentTaskWorktree`) already checks the sibling seams through the rows `{"TeardownShutdown", c.env.Teardown.Shutdown != nil}` and `{"TeardownRemove", ...}`. Card 10 neither lists that test under Edit nor adds `TeardownPush` and `TeardownLevel` rows to it.

**F17.** The set was every place in the plan that depends on card 1's tracking-ref delete keeping the upstream. Those are card 2's lines 28-30, 47-49, 58 and 66, and card 10's line 49. Each one already reads the result as "upstream set, tracking ref absent". That matches the new `DeleteRemoteTrackingRef` contract, and no card plans to use `DropRemoteTracking`. That part is covered.

Tool calls:
- Grep for `NilSeams(` callers and for `intentionallyNil`/`Teardown.Push|Level` in the plan: my context lacked the full list of NilSeams callers and the plan's other mentions.
- Grep for `DropRemoteTracking`/"tracking ref" across the plan and code: my context lacked the other cards' text that depends on the delete.
- Grep of battencli's tests for `NilSeams`/`intentionallyNil`/`Teardown.`: the first grep was cut short before it reached battencli.
- List of `Test` functions in `internal/battencli/wire_test.go`: needed the name of the test that holds the Teardown fill rows.

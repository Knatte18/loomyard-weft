```yaml
lens: forced-edits
outcome: reported
```

forced-edits lens: what each card's change forces outside the members it names.

1. BLOCKING — card 14 (new rows) must add the Drift rows to `loomshed.InterruptPolicies`, and no card does. `internal/loomshed/interruptpolicy.go:32-54` maps every row name to a policy. `internal/loomrecipe/interruptpolicy_meta_test.go:24` (`TestInterruptPolicies_MatchAssembledRows`) requires the map to cover exactly the rows `New` and `NewDarn` build. `interruptpolicy_meta_test.go:53` (`TestInterruptPolicies_MatchEngineIdentity`) requires reinvoke for every engine but Implementer. With `NameDriftBaseline`, `NameDriftAwait` and `NameDriftVerify` missing from the map, card 14's own gate fails. Fix: add `internal/loomshed#InterruptPolicies` to card 14's `**Edit:**` with all three rows mapped to reinvoke.

2. MEDIUM — card 14 names the wrong member for the sequence order. `wantSequenceOrder` is only `= deriveSequenceOrder()` (`internal/loomrecipe/sequence_test.go:62`). The outcomes come from `deriveSequenceOrder` (`sequence_test.go:64-84`), whose default case writes `Done` for every row. The Intent's "gains `Drift-Baseline` and `Drift-Await` as `skipped` entries" needs a new branch in `internal/loomrecipe#deriveSequenceOrder`, which no card lists. The edit belongs on `deriveSequenceOrder`, not the var.

3. MEDIUM — docs the decision record names are missed. The record's Docs scope names `docs/overview.md`'s execution stack, and no card touches it. Card 14 updates only the darn paragraph and the run tails; cards 1 and 7 touch only the module tree.

4. MEDIUM — card 14 leaves `internal/loomrecipe/doc.go` stale, and no card lists it. `doc.go:15` gives loom's tail as `Parent-Align → Code-Review → Describe → …`. `doc.go:17` gives darn's tail as `Code-Review → Describe` after `Darn-Write`. Both are wrong once `Drift-Verify` sits after Code-Review in loom and between `Darn-Write` and Code-Review in darn. The header text gives no `Drift-Baseline` either.

5. LOW — card 14 leaves the timing report's phase tables without the Drift rows. `internal/reportengine/timingspans.go:68-120` (`loomPhases`, `darnPhases`, `recipePhaseTables`) maps "every row of a loom run" and "every row of a darn run" to a phase. Their runs' time will fall to an unnamed phase, and the timing report will raise its no-phase findings (`internal/reportengine/timing_test.go:534`). No card lists these tables.

6. LOW — card 3's `fixedGateLists` comment pins a count. `internal/testkit/envkit/envkit.go:215` says "the eight gate keys", which turns false once `drift-verify` is added. Per the prose rule, drop the count rather than make it nine.

7. LOW — card 11 widens the `DecisionHaltPrefix` comment but leaves its package-doc twin. `internal/shedadapters/doc.go:112` still describes the prefix as the Bouncer's halt only. `internal/shedadapters/doc.go` should join card 11's `**Extends:**`.

8. LOW — card 13 depends on a test seam whose glyph I did not confirm by quarry. It names `internal/pairteardown#fakeSubstrates.teardown` and `TestTeardown_CallSequence`; the plan gate resolved both, but I did not check that the call-sequence fake records a `killCapture` slot. No defect is proven here.

Calls made:
- `internal/loomshed/interruptpolicy.go` and `internal/loomrecipe/interruptpolicy_meta_test.go`: the pre-read never covered the per-row policy table.
- `internal/loomcli/wiring_beam_test.go` and a repo grep for `Code-Bouncer` users: to check the beam wiring test and find other row-keyed tables.
- `internal/reportengine/timingspans.go` and a grep of its tests: to check whether the phase tables must cover every row.
- Grep of `TiersValue` tests, `RoleDobby` users and `stencilseed.go`: to check what the new gate key and role force.
- Grep of `internal/loomrecipe/doc.go`, `internal/loomshed/doc.go` and the loom-status and shed-recipe specs for row tails: to find prose that states the row order.
- `internal/loomrecipe/sequence_test.go:64-95`: to see how `wantSequenceOrder` is derived.

```yaml
lens: forced-edits
outcome: reported
```

Forced-edits review of the drift-rework plan (cards 1–27): edits each card's change forces that no card lists.

**Findings**

1. **BLOCKING — Card 20 (D13): the re-signed `Verdict` has a second production caller that no card lists.**
   - `internal/loomcli/wiring.go:809` calls `driftengine.Verdict(driftengine.NewPaths(location.AnchorPath()))` inside a closure in `internal/loomcli#loomCLI.wire`. Card 20 lists only `internal/loomcli#landingDeps` (`landingdeps.go:244`).
   - After the re-sign to `func Verdict(d Deps)`, that closure no longer compiles.
   - Fix: add `internal/loomcli#loomCLI.wire` to card 20's `Edit`.

2. **BLOCKING — Cards 10, 14 and 16 (D4, D8): tapestry recipe edits break `TestTapestryMotifBlock`, and no card updates its fixture.**
   - `internal/loomrecipe/blockrecipes_test.go:26-50` compares the expanded tapestry recipe, config included, against `internal/loomrecipe/testdata/tapestry-rows-before-motif.yaml`.
   - Card 10 adds `gates: [{name: drift}]` to the Webster row. Card 14 inserts the Drift-Expect row. Card 16 changes Drift-Verify's config and switches Drift-Recheck to `DriftVerify` with no `on_stuck`.
   - The fixture still holds `gate_key: drift-verify` (`:445`) and `engine: DriftRecheck` (`:577`).
   - Fix: each of those cards edits `internal/loomrecipe/testdata/tapestry-rows-before-motif.yaml` together with the recipe.

3. **BLOCKING — Cards 16 and 23 (D4, D14): the shape test's row-to-engine-type table pins Drift-Recheck to `DriftRecheck`.**
   - `internal/loomrecipe/shape_test.go:57` declares `driftRecheckType = reflect.TypeOf(&driftengine.DriftRecheck{})`, and line 104 has the row `{loomshed.NameDriftRecheck, "DriftRecheck", driftRecheckType}`.
   - Card 16 moves the row to `DriftVerify`, and card 23 deletes the type. Neither lists the shape test's table or `driftRecheckType`.
   - Card 14's new Drift-Expect row probably needs a row in that table too.

4. **BLOCKING — Card 13 (`PATTERN-shed-recipe-registry`): the new `DriftExpect` engine needs an `engineMinimalConfig` entry.**
   - `internal/shedbuild#engineMinimalConfig` says engines that need config get one, and every other engine is built with none.
   - `driftExpectEntry` requires `subject` and a `gates` list of one `expectation` entry, so building it with no config fails.
   - Card 13 does not list `engineMinimalConfig`; only card 16 lists it, for DriftVerify.

5. **MEDIUM — Card 14 (D4): adding `DriftExpect` to the recipes changes the shipped engine set.**
   - `internal/loomrecipe#TestRecipeEngines_ReportsExactlyLoomsOwnEngineSet` pins that set, but only card 16 lists the test.
   - Card 14's gate therefore fails until card 16 lands.

6. **MEDIUM — Card 16 (D4): dropping Drift-Recheck's `on_stuck` route touches tests that card 16 does not list.**
   - `internal/loomrecipe/progress_test.go:64-67` (`TestRouting_ProgressAt`) pins the Drift-Verify and Drift-Recheck edges.
   - `internal/loomrecipe/drift_test.go:27-28` (`TestDriftRows_EntryGate`) names `DriftRecheck`.
   - Card 14 lists both tests, but only for the row insertion.
   - Card 16 changes the edge and the engine, so it must list them too.

7. **LOW — Card 16: `internal/reportengine/timingspans.go:102,120` names `DriftRecheck`, unchecked.**
   - If those keys are engine names rather than row names, card 16's engine switch makes them wrong. No card lists the file. I did not confirm which it is.

8. **LOW — Cards 4 and 24: comments are left stale.**
   - The comment at `contracts/stencils/parentdirective_test.go:5` names `verify_interactive` and the Drift-Verify seat. Card 4 renames the key and card 24 removes the seat, and no card updates the comment.
   - Between cards 4 and 24, `contracts/stencils/drift/drift-template-verify.md:7` still names `verify_interactive`. This is transient, and it is gone at card 24.

9. **LOW — Overview Shared Decision `old-chairs-end-through-the-strand-table` (D4): the plan departs from the record's `runproc.EndRole` without saying so.**
   - D4 names `runproc.EndRole`, and the plan replaces it with an `EndRoles` seam over reed's strand table.
   - The overview gives a reason, but it is not recorded as a departure in the decision record, and no card states the bound of the strand stop: which strands of this worktree it may end.

Checks that came back clean need no finding.
- Every listed glyph resolves.
- The `internal/fabricengine_test` and `internal/driftcli_test` handles match their files' package clauses (`committedfile_integration_test.go:7` is `package fabricengine_test`).
- `minimalEntries` sits in `conformance_test.go`, and cards 13 and 23 list it.
- `envkit.NilSeams` skips string and non-func slice fields (`envkit.go:646-654`), so `Rows`, `FixRoute` and `JudgeSnapshotDir` need no `intentionallyNil` rows.

**Tool calls**
- `lym quarry resolve` over every existing glyph the cards cite, twice: my context did not hold quarry answers for most of them.
- `grep` for `DriftRecheck`, `drift-verify`, `Verdict` and `Defer` callers outside driftengine: my context held only the subagents' summaries of these references, not the lines.
- `grep` of fabricengine test package clauses, users of the tapestry fixture, the conformance, shape and gatequiescence tables: needed to check package clauses and to find the tests that pin recipe rows and engine types.
- Read of `blockrecipes_test.go`, `wiring.go:800-812`, the battencli seam test and `driftYAML`: to confirm how the fixture is compared and which closure encloses the second `Verdict` caller.
- `grep` of `envkit.NilSeams`'s kind switch and the beam seam markers: to decide whether new string and slice `Deps` fields need `intentionallyNil` rows.

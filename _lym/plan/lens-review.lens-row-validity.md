```yaml
lens: row-validity
outcome: reported
```

Lens: row-validity — checked whether each planned test row reaches its branch and fails when the code is wrong, and whether any test or row repeats an existing one.

- **BLOCKING — Card 8, `TestPlanWriteEntry_GateConfig` row (D6/D9):** the planned row will panic.
  The row builds the Plan-Write gate through `planContextOf` over a worktree holding a marked test.
  `seamTestFindings` then calls `index.ResolveGlyphs`, but the `env.PlanIndex` these rows use is `noFindingsPlanIndex`.
  That fake embeds a nil `planindex.Index` and says "any other method panics" (`internal/shedrecipe/fixture_test.go:72-80`; rows set `env.PlanIndex = noFindingsPlanIndex{}` at `internal/shedrecipe/entries_planwrite_test.go:303`).
  Card 3 states that the fakes "inherit the method and need no change", which is wrong for this row.
  The card needs a `noFindingsPlanIndex.ResolveGlyphs` method, listed under Create or Edit, or a different index for the row.
- **MEDIUM — Card 4, row `docs/overview.md#Modules` passes untested (D4):** the row does not reach the branch it claims.
  D4 excludes the anchor only when its unit "names [an] existing worktree file other than a `.go` or `.cs` file".
  The card's fixture holds only `internal/real/…` and `internal/dup/…` files, so there is no `docs/overview.md`.
  The token therefore parses as a member glyph, resolves `GlyphMissing` and yields `citation-symbol-missing`, and the row fails.
  The fixture must commit `docs/overview.md`.
- **MEDIUM — Card 4, D4 resolve-side ambiguity:** no row reaches the `GlyphAmbiguous` branch, a resolve answer not split by build constraint whose finding names the candidates' files.
  The only ambiguity row, `x.Foo`, reaches the `goPackageDirs` base-name branch, which is a different finding text and way forward.
  A row needs a fixture package with a member declared twice under one constraint, as card 3's `p#G` is.
- **MEDIUM — Card 7, rows added to `TestValidatePlanCmd`, `TestValidatePlanCmd_Overlay` and `TestValidatePlanCmd_ReworkReportsFindingsForFirstCardMismatch` (D7/D9):** these rows can reach `test-tier-unstated` only when the hub location carries a `gate.yaml` with tiers.
  The verb leaves `TierTags` nil without one, and these tests' fixtures run without a `gate.yaml` (`internal/loomcli/validate_test.go:603-630`, "NoGateConfigRunsTheVerb…").
  The card adds a `gate.yaml` field only to `planParityCase`.
  As written, the overlay and rework rows cannot fire.
  The `--require-approved` "does not report it" row passes vacuously and would still pass if `--require-approved` ran the check.
  Each row needs a tiered `gate.yaml` in its hub fixture.
- **MEDIUM — Card 8, D6 covering branches:** no row covers the test's file named under `Uses`, `Delete`, `Rename` or `Move`.
  No row covers a ref that is the package self glyph of the test's unit or its `_test` counterpart either.
  Of the covering forms, only the test glyph under Uses, a later card's Edit and the `_test`-unit glyph have rows.
- **MEDIUM — Card 9, D8 test-code classification:** two branches have no row.
  One is "a `Rename` new side whose old side is test code" when the name has no test prefix, for example renaming `internal/d#fixture` to `plan:internal/d#fixture2`; the `TestDo` rename row reaches only the name-prefix branch.
  The other is "an ambiguous answer is non-test code".
- **LOW — Card 8, nested-struct row:** "a seam test marking a struct nested in another … holds a card editing the nested struct" repeats the direct `internal/s#Deps` row.
  Holding is keyed on the marker glyph alone, so nesting is not a separate branch, and the fixture holds no nesting struct.
- **LOW — Card 4, the quarry-outage row's fake index:** the row needs a fake whose `ResolveGlyphs` returns an error wrapping `ErrQuarryUnavailable`, and the card does not list it.
  It is a new test type with no `Create` handle.
- **LOW — Card 4, `resolvePlainGateClosure` refusals:** the nil-`PlanIndex` and relative-`WorktreeRoot` cases are planned in `TestResolveGateSpec_ValidOneEntryPerVocabularyName`.
  Rejections already have their own tests, `TestResolveGateSpec_Rejections` and `TestResolveGateSpec_PlanGatesRequirePlanIndex` (`internal/shedrecipe/entries_gate_test.go:182,224`), and the cases belong there.
- **LOW — Card 7, done-card row:** "a webster run record as `TestNewPlanGate`'s done-card rows write it" has no helper to call.
  The done-card setup is inline in `TestNewPlanGate`'s row loop (`internal/loomshed/gates_test.go:669,705,721`), and `TestNewPlanGate` is listed only under `Uses`.
  The implementer must duplicate the setup or extract a helper, which edits a Uses member.
- **LOW — Cards 7 to 9, new top-level `TestValidatePlan_CardChecks`:** `planContextFindings` through `ValidatePlan` is already driven by `TestNewPlanGate`'s context table (`internal/loomshed/gates_test.go:660-826`).
  The D6 to D8 rows are rows of that behavior, so a separate top-level test needs a stated reason, such as the go-plan fixture shape, or the rows join that table.

Tool calls:
1. `internal/shedrecipe/fixture_test.go:60-90` and `entries_planwrite_test.go:275-340`, for the fake index behind `TestPlanWriteEntry_GateConfig`, which my context lacked.
2. A grep for go-plan tests on stub indexes, plus `planParityCase` and `gate.yaml` use in loomcli tests, to see which fixtures run on stubs.
3. `internal/loomcli/validate_test.go`'s test list and its gate.yaml-less fixture, for `TestValidatePlanCmd*`'s config.
4. The test names and tiers in `append_test.go`, `index_test.go` and `resolve_test.go`, plus the done-card setup in `gates_test.go`, to confirm the named tests and the helper card 7 relies on.
5. `TestValidateDiscussionCmd`'s env and the other `validate-discussion` test callers, for card 4's caller coverage.
6. `wireLightweight` and `TestWireLightweight_FillsThePathsWithoutLoadingAnyConfig`, for the fields the discussion verb now reads.

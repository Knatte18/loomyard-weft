# Card 3 — index-runs-vocabulary-check

**Edit:**
- `internal/planglyph#index.ValidateFormat`
- `internal/planglyph#index.ValidateRework`
- `internal/planglyph#index.ValidateDispatch`
- `internal/planglyph/doc.go`
- `internal/loomcli#loomCLI.validatePlanCmd`
- `contracts/specs/loom-plan-spec.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Create:**
- `internal/planglyph/index_test.go`
- `plan:internal/planglyph#noFabricReference` -> `type noFabricReference struct`
- `plan:internal/planglyph#noFabricReference.MatchPath` -> `func (noFabricReference) MatchPath(cmd string) (string, bool)`
- `plan:internal/planglyph#noFabricReference.MatchSpelling` -> `func (noFabricReference) MatchSpelling(cmd string) (string, bool)`
- `plan:internal/planglyph#TestIndex_AppendsFabricVocabularyName` -> `func TestIndex_AppendsFabricVocabularyName(t *testing.T)`

**Uses:**
- `plan:internal/planparser#CheckFabricVocabularyName`
- `internal/planglyph#cardIDSet`
- `internal/planglyph#minimalPlan`

**Intent:**
D9's wiring: the plan index appends `planparser.CheckFabricVocabularyName`'s findings, converted with `convertAll` (so stamped blocking), beside `card-fabric-reference`, so both plan gates (Plan-Write's and Plan-Burler-Fix's `ValidatePlan`, and PR-Rework's `ValidateReworkPlan`), `lyx loom validate-plan`'s default and `--rework` modes, and webster's dispatch report it.
- `index.ValidateFormat` and `index.ValidateRework` append the findings after the `card-fabric-reference` ones.
- `index.ValidateDispatch` appends them after `ValidateDispatch`'s answer, dropping a finding whose `Card` is in `cardIDSet(completed)`: a completed card's declared name has already landed, so a finding against it is no defect the writer can fix, matching how dispatch drops pure findings of completed cards.
`index.Validate` (the `--require-approved` mode) and `lyx webster validate`, which calls the package-level `planglyph.ValidateDispatch` directly, do not run it, as for `card-fabric-reference`.
No new gate or verb: the gates and `validate-plan` already share the index's validation (`PATTERN-gate-self-check-parity`).

`TestIndex_AppendsFabricVocabularyName` (package `planglyph`, new file `index_test.go`) builds the index with `NewIndex(noFabricReference{})`, whose two methods report no match, and a `language: none` plan from `minimalPlan` carrying one card that declares `plan:internal/x#hostBranch` -> `func hostBranch()`.
Rows, each asserting the exact count of `fabric-vocabulary-name` findings: `ValidateFormat` with no done card (one), `ValidateRework` with the told number equal to the plan's first card (one), `ValidateDispatch` with the card forthcoming (one), and `ValidateDispatch` with the card completed (none); a further row over a card declaring `plan:internal/x#plainName` reports none through `ValidateFormat`.

Docs in the same commit:
- `internal/planglyph/doc.go` names `fabric-vocabulary-name` as appended by the index's format, rework and dispatch validation beside `card-fabric-reference`, which dispatch does not run.
- `loomCLI.validatePlanCmd`'s `Long` help: the default and `--rework` modes include the `card-fabric-reference` and `fabric-vocabulary-name` checks, the latter refusing a declared name the fabric-vocabulary scan would refuse; `--require-approved` leaves both out.
- `contracts/specs/loom-plan-spec.md`: the exceptions paragraph above the numbered list gains `fabric-vocabulary-name` as a fourth exception, a `planparser` function outside both entry points that the plan index appends to its format, rework and dispatch validation; the list gains a row 41 `fabric-vocabulary-name` stating the names read (each `Create` handle's member and each `Rename` pair's `New` handle), the skip of an unparseable handle, the two rules exactly as card 2's Intent states them, with an external `_test` unit counting as its directory for the owner check, one blocking finding per name attributed to the card naming the member and `PATTERN-fabric-vocabulary` with the rename as the way forward, that it runs under any `language:`, where it is reported, and its bound (it only refuses; it misses a name spelled only in prose, such as a test row or subtest name, or built at implementation time, which the verify gate still catches; it may refuse a host-geometry helper declared in a `_test.go` file under `internal/`, which the scan admits).
  The row spells no side token: it says "a bare fabric side token".
- `cmd/lyx/testdata/test-budget.yaml` raises `internal/planglyph` by one.

**ImpactSummary:** every plan gate, `validate-plan` and webster dispatch gain one blocking check over declared names; a plan declaring no side-token or host-geometry name sees no change.

**Commit:** `3: Run fabric-vocabulary-name in the plan index`

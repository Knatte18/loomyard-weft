---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    location: _lyx/plan/03-plan-stencil-test-rules.md:10
    summary: Card 3 leaves the plan stencil's caller rule offering the file for every caller, so the stencil still tells the writer to list a _test.go caller's file under Edit, which edit-test-file now refuses
    fix: In card 3's Intent, add a bullet for the paragraph in contracts/stencils/loom/loom-template-plan.md that reads "A card that deletes or re-signs a member lists every caller of it - the file, or the member glyph whose body holds the call, on the same card for a re-sign and on the same or an earlier card for a delete." (line 161), saying that for a caller in a _test.go file the card lists only the member glyph whose body holds the call, matching card 2's caller-uncovered way forward; the existing wantAll claim on that paragraph (claims_test.go, phrases "lists every caller of it" and "on the same or an earlier card for a delete") keeps passing if those phrases are kept.
  - id: F2
    severity: LOW
    class: consistency
    location: _lyx/plan/01-edit-test-file-check.md:65
    summary: The rework parity fixture's new Edit group gets no ImpactSummary, so card-missing-field also blocks the EditTestFile row, unlike the plan-gate row, which sets one
    fix: In card 1's TestGateParity_ReworkPlanGate bullet, state that reworkParityFixture also sets the card's ImpactSummary when edit is set, so the row's only blocking finding is edit-test-file, as the TestGateParity_PlanGate bullet already does for its fixture.
  - id: F3
    severity: NIT
    class: consistency
    location: _lyx/plan/03-plan-stencil-test-rules.md:20
    summary: Card 3 misquotes the rubric sentence it drops, leaving out the word too that the rubric has after is a finding
    fix: In card 3's rubric paragraph, quote the sentence as it stands in contracts/stencils/loom/loom-rubric-plan-review.md line 63 - "An `Edit` card that lists only a test-file path while its `Intent` changes a named existing test, fake or helper is a finding too; the finding names the member glyphs to list."
---

### [MEDIUM] The stencil's caller rule still offers a test file's path

**Location:** `_lyx/plan/03-plan-stencil-test-rules.md:10`

**Issue:** `contracts/stencils/loom/loom-template-plan.md:161` reads "A card that deletes or re-signs a member lists every caller of it: the file, or the member glyph whose body holds the call, …".
Card 3 rewrites the `**Edit:**` rules at lines 149–156 and the skeleton, but leaves this paragraph alone.
For a caller in a `_test.go` file it still offers the file, which `edit-test-file` (card 1) now refuses.
Card 2 removes this offer from `caller-uncovered`'s way forward, and the record's parent decision, item 1, gives the reason: a way forward that offers a path the gate refuses contradicts the change.
The acceptance criterion "The plan stencil no longer admits a plain test-file path under `**Edit:**`" is therefore met by only some of the stencil's rules.

### [LOW] The rework parity row's Edit card has no ImpactSummary

**Location:** `_lyx/plan/01-edit-test-file-check.md:65`

**Issue:** `reworkParityFixture` (`internal/loomcli/parity_test.go:667`) writes a card with only a `Create` group and no `ImpactSummary`.
Card 1 has it add an `Edit` group when `edit` is set, but never says to set `ImpactSummary`.
`card-missing-field` blocks an `Edit` card without one, so the row reaches `stuck` for two reasons.
`wantFinding` still pins `edit-test-file`, so the row stays sound, but its fixture is inconsistent with the `TestGateParity_PlanGate` row, which the card does give an `ImpactSummary`.

### [NIT] Rubric sentence quoted without "too"

**Location:** `_lyx/plan/03-plan-stencil-test-rules.md:20`

**Issue:** The rubric's line 63 reads "… is a finding too; the finding names …", but card 3 quotes it as "… is a finding; the finding names …".
The sentence can still be identified, but the quote does not match the file.

## Rejected

- Card 2's spec row numbers 33 and 35: card 1 inserts `edit-test-file` after `verify-package-missing`, which is row 42 and the table's last row, so rows 33 (`redundant-file-target`) and 35 (`caller-uncovered`) do not shift. Not a defect.
- Member glyphs and arrows: `lyx quarry resolve` found every `Edit` glyph on cards 1 and 2, `contracts/stencils#wordingClaims` (in `contracts/stencils/claims_test.go`) included. `reworkParityFixture` (it gains `edit`) and `coverageVerbAndWay` (it gains `files`) carry the arrow. The other members change only their bodies and carry none.
- Card order: card 2's way-forward text and card 3's stencil pointer name `edit-test-file` only as text, and card 1 lands it first. No card uses something a later card creates or changes.
- Fidelity: D1, D2 and the rubric item map to card 3, D3–D5 to card 1, and the gate way-forward items 1–2 to card 2. D6 is honoured by no card touching webster. The constraints (test budget, with `internal/planparser` at its 94 cap; test economy) and every acceptance criterion map to a card. Card 2's `keepFile` field and new `RedundantFile` row are licensed by item 2. Card 1's `Glyph.UnitPath` doc rewrites are licensed by D4.
- Test economy: `TestCheckEditTestFile` calls the function directly. The `TestIndex_AppendsPlanChecks` rows pin the index's wiring and the forwarding of its `done` set. The parity rows pin both gates, the CLI, and `ParsePlan`'s canonicalization of a plain path. That matches the parity tables' existing one-row-per-appended-check convention (`FabricReference`, `DoneCardEdited`). No row duplicates another test.
- Reachability: the parse-error, path-shaped and handle skips are reached by `TestCheckEditTestFile` rows built directly as `TargetGroups`, so no upstream check rejects their input. Every listed `_test.go` input parses as a Go glyph under quarry's `checkGoUnit`. Card 2's carve-outs are reached by `TestGlyphChain_CallerCoverage`'s existing `callees/callees_external_test.go` findings and by the new `shapes/shapes_test.go` + `shapes#testHelper` row; the fixture declares `testHelper` there.
- Verify: `go test ./...` and `go test -tags integration ./...` cover every targeted package in the root module, and the section follows the template's order.
- `ignoredByGoTool` duplicates the segment rule in `internal/planglyph/deleteorder.go` and `internal/websterengine/guardstep.go`, but the leaf `planparser` cannot import either. Not a defect.
- `lyx loom validate-plan --overlay` with `{}` reports `ok: true`.

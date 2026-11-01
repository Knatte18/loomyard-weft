# Card 5 — planglyph-part-scope

**Edit:**
- `internal/planglyph#resolvePass`
- `internal/planglyph#planGatePass`
- `internal/planglyph#TestGlyphChain_CallerCoverage`

**Create:**
- `plan:internal/planglyph#TestValidateFormat_PartScope` -> `func TestValidateFormat_PartScope(t *testing.T)`

**Uses:**
- `plan:internal/planparser#PartScope`
- `internal/planglyph#TestResolvePass_ForthcomingCreateTargetExcludedFromStatus`
- `internal/planglyph/doc.go`

**Intent:** Make the resolve-backed half of the plan checks honour a part scope (D7), so a part check over `index.ValidateFormat` reports neither a dependency's export as missing nor `caller-uncovered`.

- `resolvePass`: a resolve result whose target is a self glyph whose path, read through `planparser.PathEntryPath`, is one of `plan.Part.Paths` is left out of the status policy exactly as a Create target is, so a file another part creates is no `glyph-not-found` finding.
  With `plan.Part` nil nothing changes.
- `planGatePass`: when `plan.Part` is set it reports no `caller-uncovered` finding, since that verdict rests on cards outside the part and the joined plan's `plan` gate checks it; its other findings, `delete-before-reference` included, stay.

`TestValidateFormat_PartScope` (tier0, beside `TestResolvePass_ForthcomingCreateTargetExcludedFromStatus`, building its plan the same way but calling `ValidateFormat`, the plan-gate form, rather than the dispatch form that test goes through) covers: a card using the self glyph of a file `Part.Paths` holds gives no status finding, and the same plan with `Part` nil gives `glyph-not-found`.
The `caller-uncovered` half is a row of `TestGlyphChain_CallerCoverage`, over its callees fixture, which already reaches `planGatePass` through `ValidateFormat`: a card re-signing a member with an uncovered caller gives `caller-uncovered` with `Part` nil and none with `Part` set.

**ImpactSummary:** Two passes read the optional part scope; with it nil every gate's findings are unchanged.

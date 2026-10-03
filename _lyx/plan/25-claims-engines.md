# Card 25 — claims-engines

**Edit:**
- `contracts/stencils/claims_test.go`
- `internal/loomengine/plan_test.go`
- `internal/burlerengine/template_test.go`
- `internal/burlerengine/prompt_test.go`
- `internal/shedadapters/bouncerprompt_test.go`

**Intent:** Move the raw-stencil wording checks of `loomengine`, `burlerengine` and `shedadapters` into the claims table (D12), by the same rules as card 24: raw-text `must`/`mustNot` checks become rows with a `why` (naming a section heading where the check is section-scoped), provenance-citing claims are protected (D21), generic-wording claims are dropped and itemised in the commit body, and assertions on rendered or composed output (`composePrompt`, `Fill` results, marker sets, ordering) stay where they are.
A test function left with no assertion is deleted and named in the commit body.
What this lets through, bounded per D12: a dropped generic-wording phrase can change unnoticed;
only claims without cited provenance are dropped, and each is itemised.
`loomengine/anomalybody_test.go` and `shedadapters/rubric_test.go` are left alone: the first asserts `RenderAnomalyBody`, whose text is built in Go with no stencil, and the second tests `ReadRubric` over fixture rubrics the test writes itself, so neither pins shipped stencil wording.
A raw wording check found in a test file outside this card's targets is reported as a finding, not moved silently.
`treadleengine`'s template checks stay: D12 does not name that package.

**ImpactSummary:** Test-only; three packages' raw wording checks move to `contracts/stencils`, rendered-output tests stay.

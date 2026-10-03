# Card 25 — claims-engines

**Edit:**
- `contracts/stencils/claims_test.go`
- `internal/loomengine/plan_test.go`
- `internal/loomengine/anomalybody_test.go`
- `internal/burlerengine/template_test.go`
- `internal/burlerengine/prompt_test.go`
- `internal/shedadapters/rubric_test.go`
- `internal/shedadapters/bouncerprompt_test.go`

**Intent:** Move the raw-stencil wording checks of `loomengine`, `burlerengine` and `shedadapters` into the claims table (D12), by the same rules as card 24: raw-text `must`/`mustNot` checks become rows with a `why`, provenance-citing claims are protected (D21), generic-wording claims are dropped and itemised in the commit body, and assertions on rendered or composed output (`composePrompt`, `Fill` results, marker sets, ordering) stay where they are.
A test function left with no assertion is deleted and named in the commit body.
A raw wording check in another test file of these three packages moves too.
`treadleengine`'s template checks stay: D12 does not name that package.

**ImpactSummary:** Test-only; three packages' raw wording checks move to `contracts/stencils`, rendered-output tests stay.

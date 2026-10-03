# Card 24 — claims-webster

**Edit:**
- `contracts/stencils/claims_test.go`
- `internal/websterengine/template_test.go`

**Intent:** Move `websterengine`'s raw-stencil wording checks into the claims table (D12).
Every `requireContains`/`requireNotContains` over a stencil's raw text (the accessor's bytes, before any `Fill`) becomes a claim row under that stencil's file, carrying a `why` drawn from the test's name and comment;
a test citing a crucible round or finding (for example `TestMasterTemplate_GroundsHarnessRealityAgainstInjectionRefusal`, fable-r1) keeps that provenance in its `why` and is never dropped (D21).
Claims pinning generic wording rather than behaviour (#327 §4 names `"there is no"`, `"fabric sync"`, `"AUTHORITATIVE"`) are dropped, each weighed against its `why` and listed with its reason in the commit body.
Assertions on rendered output stay in `template_test.go`: `Fill`, `RenderForkPrompt_*`, `RenderProgress_*`, ordering and marker-set tests.
A test function left with no assertion is deleted and named in the commit body.

**ImpactSummary:** Test-only; `websterengine`'s raw wording checks move to `contracts/stencils`, rendered-output tests stay.

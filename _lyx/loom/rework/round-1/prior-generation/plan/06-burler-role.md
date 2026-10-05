# Card 6 — Burler rounds load their skills and render the parent directive

**Edit:**
- `internal/burlerengine/geometry.go`
- `internal/burlerengine/engine.go`
- `internal/burlerengine/prompt.go`
- `internal/burlerengine/doc.go`
- `internal/hubgeom/hubgeom.go`
- `contracts/stencils/burler/burler-template-round-orchestrator.md`
- `internal/burlerengine/engine_test.go`
- `internal/burlerengine/prompt_test.go`
- `internal/hubgeom/hubgeom_test.go`

**Uses:**
- `internal/parentdirective#Directive`
- `internal/parentdirective#MarkerName`
- `internal/hubgeom#ResolveParent`
- `internal/burlerengine#Geometry`

**Intent:**
Move the burler round session onto D1 and D9.

`burlerengine.Geometry` gains `ParentName`.
`hubgeom.BurlerGeometry` fills it from `ResolveParent`, warning and leaving it empty when unresolvable, as `reedGeometry` does; `standalonegeom.BurlerGeometry` leaves it empty.
Every hub caller (loom's wiring and `lyx burler`'s) already builds its geometry through `BurlerGeometry`, so no wiring changes.

Skills (D2), declared beside `burlerRole`: `scribe:prose`, `scribe:code-quality`, `scribe:testing`, set on the round spec `Engine.Run` builds.

`composePrompt` fills `parentdirective.MarkerName` with `parentdirective.Directive(stencilsDir, geom.ParentName, false)`.
The opening stencil is `burler-template-round-orchestrator.md`, which carries `{{.parent_directive}}` near its top; the `burler-step-*` files and `burler-focus-directive.md` carry none.

Tests: the round prompt carries the directive with the parent's name and the no-parent variant with none; the round spec carries the D2 skill list; `BurlerGeometry` sets `ParentName` from a fixture's origin record.
`doc.go` names the skills and the directive.

**ImpactSummary:** burler's round prompt, its spec and one geometry field; `composePrompt` is package-internal and `BurlerGeometry` keeps its signature.

**Commit:** 6: burler: rounds load skills and render the parent directive

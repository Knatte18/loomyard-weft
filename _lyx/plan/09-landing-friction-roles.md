# Card 9 — Conflict, describe and friction load their skills and render the parent directive

**Edit:**
- `internal/mergeresolve/deps.go`
- `internal/mergeresolve/spec.go`
- `internal/mergeresolve/doc.go`
- `internal/landingshed/deps.go`
- `internal/landingshed/describe.go`
- `internal/landingshed/publish.go`
- `internal/landingshed/finalize.go`
- `internal/landingshed/doc.go`
- `internal/frictionengine/deps.go`
- `internal/frictionengine/spec.go`
- `internal/frictionengine/doc.go`
- `internal/loomcli/landingdeps.go`
- `internal/loomcli/wiring.go`
- `internal/loomcli/run.go`
- `internal/testkit/envkit/envkit.go`
- `contracts/stencils/landing/landing-template-conflict.md`
- `contracts/stencils/landing/landing-template-describe.md`
- `contracts/stencils/friction/friction-template-reflection.md`
- `internal/mergeresolve/mergeresolve_test.go`
- `internal/landingshed/describe_test.go`
- `internal/frictionengine/reflect_test.go`
- `internal/loomcli/landingdeps_test.go`

**Uses:**
- `plan:internal/parentdirective#Directive`
- `plan:internal/parentdirective#MarkerName`
- `internal/mergeresolve#Deps`
- `internal/landingshed#Deps`
- `internal/landingshed#DescribeInputs`
- `internal/frictionengine#Deps`

**Intent:**
Move the three remaining spawned roles of a loom run onto D1 and D9.

Parent name, told the way each package is told its stencils directory:
- `mergeresolve.Deps.ParentName`, filled by `landingshed`'s publish and finalize from a new `landingshed.Deps.ParentName`, which loom's `landingDeps` fills from the receiver's `parentName` (card 4);
- `landingshed.DescribeInputs.ParentName`, filled by `wire()`'s `DescribeSpec` closure from the same field;
- `frictionengine.Deps.ParentName`, filled by loom's `reflectFrictionRow` in `run.go` from the same field.
`envkit` sets the new fields to a fixed test parent name so kit users see a rendered directive.

Skills (D2), declared beside each role constant and set on the spec it builds:
- `conflictRole` — `scribe:prose`, `scribe:code-quality`;
- `describeRole` — `scribe:prose`;
- `frictionRole` — `scribe:prose`.

`buildConflictSpec`, `DescribeSpec` and `buildReflectionSpec` fill `parentdirective.MarkerName` with `parentdirective.Directive(stencilsDir, parentName, false)`, and each of the three stencils carries `{{.parent_directive}}` near its top.

Tests: each prompt carries the directive with the parent's name and the no-parent variant without one, and each spec carries its D2 skill list; `landingdeps_test.go` checks `landingDeps` passes the parent name through.
Each package's `doc.go` names the skills and the directive.

**ImpactSummary:** three spawn specs and their stencils, plus parent-name fields threaded from loom's wiring through landingshed into mergeresolve and into frictionengine; no behavior changes beyond the prompt text and the typed skills.

**Commit:** 9: landing, friction: conflict, describe and reflection load skills and render the parent directive

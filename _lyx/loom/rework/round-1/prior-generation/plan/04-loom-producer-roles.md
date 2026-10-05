# Card 4 — Discussion, Plan and Rework load their skills from the spec and render the parent directive

**Edit:**
- `internal/loomengine/discussion.go`
- `internal/loomengine/plan.go`
- `internal/loomengine/rework.go`
- `internal/loomengine/prompt.go`
- `contracts/stencils/loom/loom-template-discussion.md`
- `contracts/stencils/loom/loom-template-plan.md`
- `contracts/stencils/loom/loom-template-rework.md`
- `internal/loomcli/cli.go`
- `internal/loomcli/wiring.go`
- `internal/loomengine/discussion_test.go`
- `internal/loomengine/plan_test.go`
- `internal/loomengine/rework_test.go`
- `internal/loomengine/prompt_test.go`
- `internal/loomcli/wiring_test.go`
- `contracts/stencils/claims_test.go`

**Uses:**
- `internal/parentdirective#Directive`
- `internal/parentdirective#MarkerName`
- `internal/loomengine#DiscussionSpec`
- `internal/loomengine#PlanSpec`
- `internal/loomengine#ReworkSpec`
- `internal/reedengine#Geometry`

**Intent:**
Move the three loom producers onto D1 and D9.

Skills (D2), declared beside each role constant and set on the spec it builds:
- `discussionRole` — `scribe:prose`, `scribe:conversation`;
- `planRole` and `reworkRole` — `scribe:prose`, `scribe:testing`.
The "Step 0 — Load the writing skills" section leaves all three stencils, and later steps keep their numbers.

Parent directive: `DiscussionSpec`, `PlanSpec` and `ReworkSpec` each take a told `parentName string` and fill `parentdirective.MarkerName` with `parentdirective.Directive(stencilsDir, parentName, interactive)`.
`interactive` is true only for the discussion role in interactive mode (`!autonomous`), whose interview questions come from `{{.mode_rules}}`; Plan and Rework always pass false.
A directive render error fails the spec, as a pattern directive error does.
Each stencil carries `{{.parent_directive}}` where Step 0 stood, ahead of the pattern and friction directives.

Wiring: the loom receiver in `cli.go` gains a `parentName` field, which `wire()` fills from `reedGeom.ParentName` (the reed geometry it already builds through `hubgeom.ReedGeometry`), and the three spec closures pass it.
Cards 7, 9 and 10 read the same field.

Tests: each spec carries its role's skill list in order; each prompt renders the directive with the parent's name, and the no-parent variant with an empty name; the interactive discussion prompt omits the operator-ban line and the autonomous one carries it.
`prompt_test.go`'s Step 0 skill-name assertions and `claims_test.go`'s "Step 0 loads the prose and testing skills" claim are reworded messages: replace them with assertions that the skill names are absent from the stencil text and present on the spec, and keep `claims_test.go`'s `scribe:conversation` absence claim for the plan stencil.
`wiring_test.go`'s spec-shape tests gain the parent name and skill list.

**ImpactSummary:** three loom producer specs and their stencils, plus one loom receiver field; the spec signatures change at their one caller, `internal/loomcli/wiring.go`.

**Commit:** 4: loom: producers load skills from the spec and render the parent directive

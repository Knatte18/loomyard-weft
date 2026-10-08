# Card 20 — Conflict, Describe, reflection and driver prompts carry the edit directive

**Edit:**
- `internal/mergeresolve#buildConflictSpec`
- `internal/landingshed#DescribeSpec`
- `internal/frictionengine#buildReflectionSpec`
- `internal/loomcli#driverPrompt`
- `contracts/stencils/landing/landing-template-conflict.md`
- `contracts/stencils/landing/landing-template-describe.md`
- `contracts/stencils/friction/friction-template-reflection.md`
- `contracts/stencils/shed/shed-template-driver.md`
- `internal/mergeresolve/mergeresolve_test.go`
- `internal/landingshed/describe_test.go`
- `internal/frictionengine/reflect_test.go`
- `internal/loomcli/driverprompt_test.go`
- `internal/landingshed/finalize_integration_test.go`
- `internal/mergeresolve/seam_enforcement_test.go`
- `internal/landingshed/seam_enforcement_test.go`
- `internal/frictionengine/seam_enforcement_test.go`

**Uses:**
- `internal/editdirective#Directive`
- `internal/editdirective#MarkerName`

**Intent:**
Decision 14 for the remaining spawned roles: `conflict`, `describe`, `friction` (the reflection role) and `driver`.
The describer and the reflection role are included, since each writes its own artifact.

Each of the four stencils gains a `{{.edit_directive}}` line directly below `{{.parent_directive}}`, and its header comment names the marker among the required ones.
`buildConflictSpec` reads the directive from `deps.StencilsDir`, `DescribeSpec` from `in.StencilsDir`, `buildReflectionSpec` from `deps.StencilsDir` and `driverPrompt` from its `stencilsDir`.
Each fills `editdirective.MarkerName` beside `parentdirective.MarkerName` and returns a read failure as it does the parent directive's.
The three seam-enforcement allowlists of `mergeresolve`, `landingshed` and `frictionengine` gain `internal/editdirective`.

Tests:
- `mergeresolve_test.go`, `describe_test.go` and `reflect_test.go` seed the stencils through `stencilkit.SeedInto`, so `edit-directive` is present, and then overwrite the role's own stencil with a hand-written fixture (`conflictStencilFixture`, `describeStencilFixture`, `reflectionStencilFixture`).
  Each fixture gains a `{{.edit_directive}}` line below its `{{.parent_directive}}` line, since `stencil.Fill` drops a value whose marker the template lacks, and each test asserts the rendered prompt contains the directive's `Edit or Write` sentence.
- `landingshed/finalize_integration_test.go`'s `finalizeConflictStencilFixture`, which its comment says mirrors mergeresolve's fixture, gains the same line.
- `driverprompt_test.go` renders the seeded driver stencil unchanged and asserts the same sentence.

**ImpactSummary:** Four renderers in four packages fill one more required marker; three import allowlists gain one entry.

**Commit:** 20: landing, friction, driver: their prompts carry the edit directive

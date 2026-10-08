# Card 18 — Treadle prompts carry the edit directive

**Edit:**
- `internal/treadleengine#runTargeting`
- `internal/treadleengine#runCircling`
- `internal/treadleengine#runMilestone`
- `contracts/stencils/treadle/treadle-template-targeting.md`
- `contracts/stencils/treadle/treadle-template-judge-circling.md`
- `contracts/stencils/treadle/treadle-template-judge-milestone.md`
- `internal/treadleengine/template_test.go`
- `internal/treadleengine/judge_test.go`
- `internal/treadleengine/seam_enforcement_test.go`
- `internal/treadleengine/doc.go`

**Uses:**
- `plan:internal/editdirective#Directive`
- `plan:internal/editdirective#MarkerName`

**Intent:**
Decision 14 for `treadle-targeting` and both `treadle-judge` stencils.

Each of the three stencils gains a `{{.edit_directive}}` line directly below `{{.parent_directive}}`, and its header comment names it as required.
`runTargeting` reads the directive from its `stencilsDir`, and `runCircling` and `runMilestone` from `in.StencilsDir`.
Each fills `editdirective.MarkerName` beside `parentdirective.MarkerName` and returns a read failure as it does the parent directive's.
`doc.go`'s sentence about `{{.parent_directive}}` near each opening stencil's top names `{{.edit_directive}}` beside it.
`seam_enforcement_test.go`'s import allowlist gains `internal/editdirective`, and its header list of allowed imports names it.

Tests:
- `template_test.go`'s hand-built values maps gain an `edit_directive` value.
- `judge_test.go` asserts the circling and milestone prompts it renders contain the directive's `Edit or Write` sentence.

**ImpactSummary:** Three treadle renderers fill one more required marker; the package's import allowlist gains one entry.

**Commit:** 18: treadle: targeting and judge prompts carry the edit directive

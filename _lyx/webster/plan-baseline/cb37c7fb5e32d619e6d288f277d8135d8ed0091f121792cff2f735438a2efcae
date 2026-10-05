# Card 8 — Treadle targeting, judge and triage load their skill and render the parent directive

**Edit:**
- `internal/treadleengine/engine.go`
- `internal/treadleengine/run.go`
- `internal/treadleengine/judge.go`
- `internal/treadleengine/targeting.go`
- `internal/treadleengine/doc.go`
- `contracts/stencils/treadle/treadle-template-targeting.md`
- `contracts/stencils/treadle/treadle-template-judge-circling.md`
- `contracts/stencils/treadle/treadle-template-judge-milestone.md`
- `contracts/stencils/treadle/treadle-template-triage.md`
- `internal/treadleengine/judge_test.go`
- `internal/treadleengine/engine_test.go`

**Uses:**
- `plan:internal/parentdirective#Directive`
- `plan:internal/parentdirective#MarkerName`
- `internal/treadleengine#Options`

**Intent:**
Move treadle's three spawned roles onto D1 and D9.

`treadleengine.Options` gains `ParentName`, told by the caller like `StencilsDir`, and the engine threads it to `judgeInputs` and the triage and targeting calls as it does the stencils directory.

Skills (D2), declared beside `judgeRole`, `triageRole` and `targetingRole`: `scribe:prose` for each, set on the specs `runJudgeCall`, `runTriage` and `runTargeting` build.

Each of the four opening stencils carries `{{.parent_directive}}` near its top, filled with `parentdirective.Directive(stencilsDir, parentName, false)` at the same call that fills its other markers; a directive render error takes the same fallback path a stencil read error takes there today.

Tests: the judge (circling and milestone), triage and targeting prompts carry the directive with the parent's name and the no-parent variant without one, and each spec carries `scribe:prose`, as rows of the existing per-call tests.
`doc.go` names the skill and the directive.

**ImpactSummary:** treadle's four spawn renders and one Options field; treadleengine has no production caller outside its own package, so only its tests see the change.

**Commit:** 8: treadle: spawned roles load their skill and render the parent directive

# Card 16 — Discussion, Plan and Rework prompts carry the edit directive

**Edit:**
- `internal/loomengine#composePrompt`
- `internal/loomengine#composePlanPrompt`
- `internal/loomengine#composeReworkPrompt`
- `contracts/stencils/loom/loom-template-discussion.md`
- `contracts/stencils/loom/loom-template-plan.md`
- `contracts/stencils/loom/loom-template-rework.md`
- `internal/loomengine/plan_test.go`

**Uses:**
- `internal/editdirective#Directive`
- `internal/editdirective#MarkerName`

**Intent:**
Decision 14 for the three loom writer roles (`discussion`, `plan`, `rework` in `roleOpeningStencils`).

Each opening stencil gains a `{{.edit_directive}}` line directly below its `{{.parent_directive}}` line.
Its header comment names the marker as required and rendered by `internal/editdirective`.

`composePrompt`, `composePlanPrompt` and `composeReworkPrompt` each call `editdirective.Directive(stencilsDir)` after reading their template, return its error wrapped with the composer's own prefix, and fill `editdirective.MarkerName`.
The marker stays out of the optional-names slice, so a prompt rendered without the directive fails at composition rather than silently lacking the rule.
No signature changes: each composer already holds `stencilsDir`.

Tests: `plan_test.go`'s composition test asserts the rendered Plan prompt contains the directive's `Edit or Write` sentence when the repo has no `PATTERN.md` (pattern directive empty), which pins the acceptance criterion that the rule renders unconditionally.
The discussion and rework composition tests render the seeded stencils through the same composers and keep passing.

**ImpactSummary:** Three loom composers each read one more stencil and fill one more required marker; their prompts gain the edit section.

**Commit:** 16: loom: Discussion, Plan and Rework prompts carry the edit directive

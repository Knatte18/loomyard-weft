# Card 17 — Burler and Bouncer prompts carry the edit directive

**Edit:**
- `internal/burlerengine#composePrompt`
- `internal/shedadapters#Bouncer.runSeedSpawn`
- `internal/shedadapters#Bouncer.judgeCall`
- `contracts/stencils/burler/burler-template-review-orchestrator.md`
- `contracts/stencils/burler/burler-template-fix-orchestrator.md`
- `contracts/stencils/bouncer/bouncer-template-judge.md`
- `contracts/stencils/bouncer/bouncer-template-seed.md`
- `internal/burlerengine/template_test.go`
- `internal/burlerengine/prompt_test.go`
- `internal/shedadapters/bouncer_judge_test.go`
- `internal/shedadapters/bouncer_seed_test.go`

**Uses:**
- `plan:internal/editdirective#Directive`
- `plan:internal/editdirective#MarkerName`

**Intent:**
Decision 14 for `burler-review`, `burler-fix`, `bouncer-judge` and `bouncer-seed`.
Reviewers and judges are included: each writes at least its own artifact, and the rule constrains the tool, never the write surface.

Each of the four opening stencils gains a `{{.edit_directive}}` line directly below `{{.parent_directive}}`, and its header comment names it as required.
`burlerengine.composePrompt` reads the directive once and fills `editdirective.MarkerName` in both the reviewer's and the fixer's fill.
`Bouncer.runSeedSpawn` and `Bouncer.judgeCall` each read it from `b.cfg.StencilsDir` and fill it.
Every renderer returns a read failure as the error it already returns for the parent directive's.
The marker is never optional.

Tests:
- `burlerengine/template_test.go`'s hand-built values maps gain an `edit_directive` value.
- `prompt_test.go` asserts both rendered orchestrator prompts contain the directive's `Edit or Write` sentence.
- `bouncer_judge_test.go` and `bouncer_seed_test.go` assert the same of the judge and seed prompts they already render.

**ImpactSummary:** Burler's prompt composer and two Bouncer spawn paths fill one more required marker; four review-segment prompts gain the edit section.

**Commit:** 17: burler, bouncer: review and judge prompts carry the edit directive

# Card 7 — sampler-vocabulary

**Create:**
- `plan:internal/loomcli#samplerTacklerRecipe` -> `const samplerTacklerRecipe = "sampler: an interactive task run (one agent session working with the operator, its optional review, description and landing)"`

**Edit:**
- `internal/battencli#childRecipeSupported`
- `internal/battencli#unsupportedChildRecipeRefusal`
- `internal/battencli#TestWire_WriteSeedRefusesANonLoomChildBeforeTouchingTheWorktree`
- `internal/battencli#TestPrepareRunSettings`
- `internal/boardengine#normalizeTask`
- `internal/boardengine_test#TestNewTask`
- `internal/runprofile/template.yaml`
- `internal/runprofile/doc.go`
- `internal/runprofile_test#TestTemplateLadder`
- `internal/loomcli#tacklerFor`
- `internal/loomcli#arbiterRecipeDescription`
- `internal/loomcli#TestTacklerFor`
- `internal/loomcli#TestRecipeKeyedWiring`
- `contracts/specs/landing-fixer-report-spec.md`
- `contracts/specs/loom-status-spec.md`
- `contracts/stencils/landing/landing-template-pr-review.md`
- `internal/gateslot/doc.go`
- `contracts/specs/loom-final-summary-spec.md`
- `contracts/stencils/orch/orch-template-role.md`
- `contracts/stencils/loom/loom-rubric-code-review.md`
- `contracts/stencils#LoomRubricCodeReview`
- `internal/landingshed#EscalateStopPrefix`
- `internal/landingshed#fixerOutcome.haltReason`
- `internal/landingshed/handback.go`
- `internal/landingshed#Deps`
- `internal/landingshed#Finalize.gateAndFix`
- `internal/landingshed#ParentAlign.Call`
- `internal/landingshed/doc.go`
- `internal/arbiterstop#Eligibility`
- `internal/arbiterstop/doc.go`
- `internal/sheddriver/doc.go`
- `internal/battenshed#ErrUnsupportedChildRecipe`
- `internal/battenshed#innerRunProducer.binaryRestep`
- `internal/battenshed/doc.go`
- `internal/battencli#battenCLI.wire`
- `internal/battencli#restepSkips`
- `internal/battencli#childSeedParams`
- `internal/battenshed#InnerRunDeps`
- `internal/loomcli#loomArbiterSeam`
- `internal/loomcli#loomCLI.specFor`
- `internal/loomcli/settledguard.go`
- `internal/loomcli/subjects.go`
- `docs/overview.md`

**Uses:**
- `internal/boardengine/doc.go`
- `internal/loomrecipe#HasEngine`
- `internal/battencli#TestWire_EverySeamFilled`
- `internal/loomcli#TestLandingDeps_EveryFieldPopulated`
- `internal/loomcli#TestWire_BeamRecipe`
- `internal/loomcli#TestWire_EverySeamFilled`
- `internal/testkit/envkit#TestFullEnv_FillsEverySeam`
- `internal/testkit/envkit#TestNilSeams_ReportsExactPaths`
- `internal/testkit/envkit#TestNilSeams_SkipsNilLegalPaths`

**Intent:**
D2: the recipe name reaches the seeding, board, profile and description sites.
`childRecipeSupported` admits `shedrun.RecipeSampler` beside tapestry, darn and pawl, and `unsupportedChildRecipeRefusal` names the four recipes `lym loom start` serves and says a Board task's type must be tapestry, darn, sampler or empty; `TestWire_WriteSeedRefusesANonLoomChildBeforeTouchingTheWorktree` iterates the four admitted recipes in both of its loops, so a `recipe: sampler` task reaches the absent-pair refusal past the recipe checks (D2 acceptance).
The board already accepts any task recipe and refuses any project recipe but `pawl`; `normalizeTask`'s refusal of a task carrying `pawl` now says a task's recipe is tapestry, darn or sampler, and `TestNewTask`'s `task pawl` row expects that text; a project with `recipe: sampler` is refused by `checkProjectFields`' branch for every recipe but `pawl`, which `TestNewTask`'s existing `project tapestry` row already reaches, so no row is added (D2 acceptance).
`runprofile/template.yaml`'s header names sampler among the recipes, and the `speed`, `light`, `solid` and `wide` profiles each gain a `sampler` block whose `loom` mapping repeats that profile's `darn` block; `speed`'s carries no `sampler` key, leaving `code_review` false, and `light`'s, `solid`'s and `wide`'s carry `sampler: {code_review: true}`; `defaults` gets no sampler entry, so a task naming no profile runs on the hub's config with the perch off.
`TestTemplateLadder` asserts each ladder profile's sampler block carries its darn block's `loom` values and `sampler.code_review` true exactly for light, solid and wide, and that `Default("sampler")` is empty; `TestPrepareRunSettings` gains one row over the template profiles: recipe `sampler` with profile `light` writes a `sampler` document `code_review: true` and the light `loom` values (D2 acceptance); a sampler task naming no profile takes the existing "no profile and no default for the recipe" branch that test already reaches, and `TestTemplateLadder`'s empty `Default("sampler")` pins that sampler has no default.
`runprofile/doc.go` names sampler among the recipes and says the sampler blocks carry darn's perch values with the review on from light up.
`tacklerFor` returns `samplerTacklerRecipe` for a recipe with a `SamplerSession` row, judged by `loomrecipe.HasEngine` before the Plan-Write branch, and `arbiterRecipeDescription` describes such a run as a task's result made with the operator in one interactive session, then its optional review and landing, with the repair verbs; `TestTacklerFor` and `TestRecipeKeyedWiring` gain a sampler row (tree scope true, no plan, the sampler fragment).
The doc lines that state a rule for "tapestry or darn" runs that a sampler run now falls under gain sampler: the fixer escalate and hand-back sentences of `landing-fixer-report-spec.md` and `loom-status-spec.md`, the leading comment of `landing-template-pr-review.md`, the Code-Burler-Fix, PR-Review and Finalize rows of `gateslot/doc.go`'s site table, the Describe sentence of `loom-final-summary-spec.md`, the orch role's two lines (the run batten drives as the entry's recipe says, and the fixer escalate that reaches the orch through the arbiter), the Code-Review rubric's leading comment and its `LoomRubricCodeReview` doc, and the escalate, hand-back, admitted-recipe and arbiter-seam comments of `EscalateStopPrefix`, `fixerOutcome.haltReason`, `EscalateHalt`, the `EscalateJudgedFirst` field of landingshed's `Deps`, `Finalize.gateAndFix`, `ParentAlign.Call`, `handback.go`'s file comment, `arbiterstop.Eligibility`, `ErrUnsupportedChildRecipe`, `innerRunProducer.binaryRestep`, battencli's `battenCLI.wire` and `restepSkips`, loomcli's `loomArbiterSeam`, `specFor`'s hand-back comment, and the file comments of `settledguard.go` and `subjects.go`, plus the package docs of landingshed, arbiterstop, sheddriver and battenshed and `docs/overview.md`'s batten lines naming the admitted recipes.
The `childSeedParams` doc lists sampler among the recipes that declare `params.parent`, since the shared loom bootstrap writes it for every loom recipe, and the `HandBack` field doc of `InnerRunDeps` says a tapestry, darn or sampler escalate halt; a struct field is not a glyph of its own, so the struct carries the target.
Comments that describe only the tapestry and darn recipes' own embedded files stay: `IsFinalizeRow`, `BouncerRunSubdir`, `loomrecipe/doc.go`'s role-prefix pin and loomcli's subject-key sentence remain true.

**ImpactSummary:** widens five recipe lists and their tests; no behaviour changes for tapestry, darn or pawl runs, and the board's only change is one refusal's wording; beside the doc comments, the sweep edits agent-facing text: the orch role stencil's two lines and the PR-Review stencil's leading comment.

**Commit:** 7: sampler reaches batten's seeding, the board, the run profiles and the run descriptions

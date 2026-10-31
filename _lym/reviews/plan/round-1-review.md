---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:repo-invariants
    location: _lym/plan/05-sampler-registry-entry.md
    summary: Card 5 registers SamplerSession in the shedrecipe registry, but TestCoverageGuard fails until card 6 adds the sampler recipe to loomrecipe.RecipeEngines, so card 5 cannot pass its own card gate.
    fix: "Move the recipe file and its reach into RecipeEngines from card 6 to card 5. In 05-sampler-registry-entry.md, add under Create `contracts/recipes/sampler-recipe.yaml` and `plan:contracts/recipes#SamplerRecipe` -> `var SamplerRecipe []byte`; add under Edit `internal/loomrecipe#RecipeEngines` and `internal/loomrecipe#TestRecipeEngines_ReportsExactlyLoomsOwnEngineSet`; add under Extends `contracts/recipes/recipes.go`; and add an Intent sentence: so that TestCoverageGuard holds at this card, the card also writes sampler-recipe.yaml with the rows card 6 describes, embeds it in recipes.go as SamplerRecipe, and adds it to RecipeEngines' source list, with TestRecipeEngines_ReportsExactlyLoomsOwnEngineSet gaining SamplerSession. In 06-sampler-recipe.md, remove those same Create, Edit and Extends lines, add `plan:contracts/recipes#SamplerRecipe` under Uses, and trim the Intent sentences about embedding the recipe, RecipeEngines and TestRecipeEngines_ReportsExactlyLoomsOwnEngineSet. The recipe-content description stays in card 6's Intent; card 5 points at it. Checked with validate-plan --overlay: no new blocking finding."
  - id: F2
    severity: MEDIUM
    class: consistency
    origin: lens:repo-invariants
    location: _lym/plan/08-rows-keyed-wiring.md
    summary: Card 8 makes codeBudgetZero and rerouteReviewed also read the perch rows' entry_gate. The landingshed Deps field docs for CodeReviewOff and RerouteReviewed still say each reads only the Code-Perch budget, and no card lists them.
    fix: "In 08-rows-keyed-wiring.md, add `internal/landingshed#Deps` under Edit. Add under Uses `internal/landingshed/doc.go`, `internal/battencli#TestWire_EverySeamFilled`, `internal/loomcli#TestLandingDeps_EveryFieldPopulated`, `internal/loomcli#TestWire_BeamRecipe`, `internal/loomcli#TestWire_EverySeamFilled`, `internal/testkit/envkit#TestFullEnv_FillsEverySeam`, `internal/testkit/envkit#TestNilSeams_ReportsExactPaths` and `internal/testkit/envkit#TestNilSeams_SkipsNilLegalPaths`. Add an Intent sentence: the CodeReviewOff and RerouteReviewed field docs say the perch reads as off when its budget is 0 or when the Code-Review rows' entry gate leaves them out. Checked with validate-plan --overlay: no new blocking finding."
  - id: F3
    severity: MEDIUM
    class: design
    origin: handler
    location: _lym/plan/03-sampler-stencil.md
    summary: TestSessionSpec asserts the stencil's static prose. It checks that the prompt names lym loom sampler-done, lym loom decision add, --by operator, a note through lym board, tmux -L and the sandbox hub, and holds neither git push nor gh pr. The rubric flags any planned assertion on a stencil's static prose.
    fix: "In 03-sampler-stencil.md's Intent, cut from TestSessionSpec the assertions on the stencil's fixed wording: the clause from `that it names lym loom sampler-done` through `that it holds neither git push nor gh pr`. Keep the assertions on the spec's flags, role, segment, skills and model choice, on the rendered interactive parent directive and edit directive, and on the filled record path and slug. Keep the Slug and agent refusal rows. The D8 and D9 wording rests on the stencil text the card writes, and the stencil-wide scans the card names cover it. No target changes."
  - id: F4
    severity: LOW
    class: consistency
    origin: lens:affected-set
    location: _lym/plan/07-sampler-vocabulary.md
    summary: Card 7's Intent edits handback.go's file comment, which names only tapestry and darn escalate halts. No target covers that line, because the card lists only the EscalateHalt member of the file.
    fix: "In 07-sampler-vocabulary.md, under Edit, replace `internal/landingshed#EscalateHalt` with the file target `internal/landingshed/handback.go`. The file target covers both EscalateHalt and the file comment, and the redundant-file-target check refuses listing the file beside its member. Checked with validate-plan --overlay: no new blocking finding."
  - id: F5
    severity: LOW
    class: consistency
    origin: lens:repo-invariants
    location: _lym/plan/12-operator-wait-rows.md
    summary: The step_idle_timeout_min comment in internal/loomengine/template.yaml says the loop kills any step with no activity for the window. After card 12, a step at an operator-wait row is never killed, and no card lists the comment.
    fix: "In 12-operator-wait-rows.md, add `internal/loomengine/template.yaml` under Edit. Add an Intent sentence: the step_idle_timeout_min comment says a step at a row waiting on the operator (Beam-Await, PR-Review, Sampler-Session) reads as active and is never killed. Checked with validate-plan --overlay: no new blocking finding."
  - id: F6
    severity: LOW
    class: consistency
    origin: lens:repo-invariants
    location: _lym/plan/06-sampler-recipe.md
    summary: RowNames' doc lists the tapestry, darn, batten and pawl recipes and says any other name is refused, which is false once card 6 embeds sampler. Card 6 edits Rows, which carries the same wording, but not RowNames.
    fix: "In 06-sampler-recipe.md, add `internal/loomrecipe#RowNames` under Edit. Extend the Intent so that the Rows and RowNames docs both name shedrun.RecipeSampler as the sampler recipe. Checked with validate-plan --overlay: no new blocking finding."
  - id: F7
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/plan/09-sampler-wiring.md
    summary: Card 9's ImpactSummary claims new sampler-only branches in wire, wireLanding and buildLoomShed, while its Intent says buildLoomShed needs no change and is not edited.
    fix: "In 09-sampler-wiring.md, rewrite the ImpactSummary to name only wire and wireLanding: new sampler-only wiring branches in wire and wireLanding, taken only for a recipe with a SamplerSession row; tapestry, darn and pawl wiring is unchanged."
---

### [BLOCKING] Card 5 cannot pass its gate before card 6 reaches the new engine

**Location:** `_lym/plan/05-sampler-registry-entry.md`

**Issue:** `internal/shedrecipe/coverage_guard_test.go:51` `TestCoverageGuard` (tier0, in shedrecipe's external test package) requires every registered engine to be reached by `loomrecipe.RecipeEngines()` or `battenrecipe.RecipeEngines()`, or to be allowlisted.
`loomrecipe.RecipeEngines` parses a fixed list of embedded sources (`internal/loomrecipe/names.go:28`), and card 6 adds the sampler recipe to that list.
Card 5 adds `SamplerSession` to the registry one card earlier, so the shedrecipe card gate fails at card 5, and neither card lists the guard.
`RecipeEngines` only parses the file and never builds it against the registry, so the recipe file and its `RecipeEngines` reach can move into card 5.
Allowlisting the engine for one card and removing it in card 6 would also pass, but the guard's drift subtest forces the second edit, so moving the recipe is the cleaner fix.
The combined overlay of this fix and the target fixes below reports no blocking finding.

### [MEDIUM] Landing seam docs still read the perch as budget-only

**Location:** `_lym/plan/08-rows-keyed-wiring.md`

**Issue:** `internal/landingshed/deps.go:320-326` documents `CodeReviewOff` as "whether the Code-Perch's effective review budget is 0".
It documents `RerouteReviewed` as, for loom, "the Code-Review perch's effective budget being above 0".
Card 8 makes both loom seams also read the perch off when the Code-Bouncer row's entry gate does not hold (D3), so on a sampler run with `code_review` false both docs become false.
Card 7 lists `landingshed#Deps`, but its Intent touches only the `EscalateJudgedFirst` field.
The overlay needs the Env-reflecting seam tests and landingshed's `doc.go` under Uses to clear `seam-test-unlisted` and `doc-go-unlisted`, and the fix lists them.

### [MEDIUM] TestSessionSpec asserts the stencil's static prose

**Location:** `_lym/plan/03-sampler-stencil.md`

**Issue:** The rubric's test-economy rule names "a card that plans an assertion on a stencil's static prose" as a finding.
`TestSessionSpec` asserts that the rendered prompt names `lym loom sampler-done`, `lym loom decision add`, `--by operator`, a note through `lym board`, `tmux -L` and the sandbox hub, and that it holds neither `git push` nor `gh pr`.
Each of these is fixed wording in `sampler-template-agent.md`, not a value the spec builder computes.
The D8 and D9 acceptance lines name what the stencil carries, and the stencil the card writes carries it.
The scans the card already names cover the denied-step and ask-the-operator rules for every registered stencil: `TestStencils_NoUnboundedAskOperator` and the other opening-stencil scans.
The assertions that test `SessionSpec`'s own behaviour stay.

### [LOW] handback.go's file comment has no target

**Location:** `_lym/plan/07-sampler-vocabulary.md`

**Issue:** `internal/landingshed/handback.go:1` reads "a tapestry or darn escalate halt that goes to the arbiter first".
Card 7's Intent lists "`handback.go`'s file comment" among its edits, but the Edit list holds only `internal/landingshed#EscalateHalt` (line 29), whose span does not include line 1.

### [LOW] The idle-timeout template comment goes stale

**Location:** `_lym/plan/12-operator-wait-rows.md`

**Issue:** `internal/loomengine/template.yaml:26` says a `--until-stop` step showing no activity for the window is killed by the loop.
Card 12's Bound says the docs will state the exemption, but the template comment, which `lym config loom` shows the operator, is not listed.

### [LOW] RowNames' doc goes stale

**Location:** `_lym/plan/06-sampler-recipe.md`

**Issue:** `internal/loomrecipe/names.go:55-56` enumerates the four recipes `RowNames` accepts and says any other name is refused.
Once `embeddedRecipeSource` maps sampler, `RowNames(sampler)` succeeds, so the doc is false, and card 6 lists only `Rows`.

### [LOW] Card 9's ImpactSummary disagrees with its Intent

**Location:** `_lym/plan/09-sampler-wiring.md`

**Issue:** The Intent says "`buildLoomShed` needs no change of its own, so it is not edited", but the ImpactSummary names buildLoomShed among three functions that gain sampler-only branches.

## Rejected

- affected-set, `internal/reportengine#runRecipes` and its phase table: the decision record's scope names the seed vocabulary, `lym shed`, the board, the profiles, and the docs that name the loom recipes, not run reports. The report docs stay true while sampler runs are left out ("seeded to a recipe other than tapestry or darn, is left out silently"). Adding a sampler phase table would be scope the answer key does not license.
- affected-set, `internal/loomcli/drivewiring.go:129`: that line is `loomArbiterSeam`'s doc comment, which card 7 already lists under Edit and names in its Intent.
- guard-reach, card 8's `code_review: "yes"` row: the row unit-tests the seam's propagation of an entry-gate evaluation error. The file can change between `wire`'s load and Finalize's call in a later step, so the input is not foreclosed, and the error branch is a propagated error, not a prescribed refusal.

---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: scope
    location: _lym/plan/05-sampler-registry-entry.md:Edit
    summary: Card 5 registers SamplerSession but lists no edit to shedrecipe TestNames, whose ShipsExpectedEntries subtest pins the exact registry name list, so the card gate fails
    fix: In 05-sampler-registry-entry.md add `- \`internal/shedrecipe#TestNames\`` to the **Edit:** list, and add one Intent sentence saying `TestNames`' `ShipsExpectedEntries` subtest gains `"SamplerSession"` in its sorted expected list, between `"RecordWrite"` and `"SeedChild"`. The added target passes `lym loom validate-plan --overlay`.
  - id: F2
    severity: LOW
    class: scope
    location: _lym/plan/06-sampler-recipe.md:Edit
    summary: Card 6 makes the AmendmentTarget doc false by admitting sampler but lists no target for that function
    fix: In 06-sampler-recipe.md add `- \`internal/loomrecipe#AmendmentTarget\`` to the **Edit:** list, and extend the Intent sentence on the Rows and RowNames docs so that the `AmendmentTarget` doc also names `shedrun.RecipeSampler` in its list of recipe names. The added target passes `lym loom validate-plan --overlay`.
  - id: F3
    severity: LOW
    class: scope
    location: _lym/plan/07-sampler-vocabulary.md:Edit
    summary: Card 7 sweeps the tapestry-or-darn doc lines but misses the childSeedParams recipe list and the InnerRunDeps HandBack field doc
    fix: In 07-sampler-vocabulary.md add `- \`internal/battencli#childSeedParams\`` and `- \`internal/battenshed#InnerRunDeps\`` to the **Edit:** list. In the Intent's doc sweep add that the `childSeedParams` doc lists sampler among the recipes that declare `params.parent` (the shared loom bootstrap writes it for every loom recipe), and that the `HandBack` field doc of `InnerRunDeps` says tapestry, darn or sampler escalate halt. A struct field is not a glyph of its own, so the struct carries the target. The added targets pass `lym loom validate-plan --overlay`.
  - id: F4
    severity: MEDIUM
    class: consistency
    location: _lym/plan/03-sampler-stencil.md:Intent
    summary: Card 3 has the stencil name docs/sandbox-howto.md, a bare path that TestStencils_NoBareCrossRepoCitations refuses, and the card lists no allowlist entry
    fix: In 03-sampler-stencil.md reword the D9 sentence of the Intent so the stencil names the sandbox hub on the machine, on lym's own repository, without the path `docs/sandbox-howto.md`, and state that the stencil carries no `docs/`, `contracts/` or `internal/` path because `TestStencils_NoBareCrossRepoCitations` refuses one (the path also does not exist in the other repositories a sampler run may work in). Keep `docs/sandbox-howto.md` under **Uses:** as the source the writer reads.
  - id: F5
    severity: LOW
    class: consistency
    location: _lym/plan/02-sampler-record.md:ImpactSummary
    summary: ImpactSummary lines of cards 2, 5, 6, 7, 8 and 11 do not match the functions their Intent edits
    fix: "Rewrite the ImpactSummary lines. Card 2 (02-sampler-record.md): TestPathConstructors gains two rows, one per accessor, not one. Card 5 (05-sampler-registry-entry.md): name the production change to checkLoomContract's engine-choice text beside the TestBuild_LoomContract strings, and say that widening LoomBuildEngines also reaches loomrecipe verifyGatedRows, which card 6 narrows. Card 6 (06-sampler-recipe.md): add that InterruptPolicies gains a Sampler-Session entry. Card 7 (07-sampler-vocabulary.md): add that the sweep edits agent-facing text, the orch role stencil's two lines and the PR-Review stencil comment, beside the doc comments. Card 8 (08-rows-keyed-wiring.md): codeBudgetZero has two production callers (landingDeps and rerouteReviewed), rerouteReviewed one; add that darnDeps gains a recipe argument, that wireDarn loses its budgets parameter, and that shedbuild RowEntryGate and loomrecipe CodePerchGate are new exported functions. Card 11 (11-decision-sampler-caller.md): the change also edits loomcli/doc.go, so it does not stay inside decision.go and its tests."
---

### [BLOCKING] Card 5 leaves the registry name pin failing at its own gate

**Location:** `_lym/plan/05-sampler-registry-entry.md` (the **Edit:** list); the pin is `internal/shedrecipe/registry_test.go`, `TestNames`, subtest `ShipsExpectedEntries`.
**Issue:** Card 5 adds `SamplerSession` to the one registry map. `TestNames/ShipsExpectedEntries` compares `Names()` with a hard-coded sorted list of every registered engine and fails on any length or element difference. The plan never names `TestNames` (card 5 lists `minimalEntries`, which `TestEveryEntry_RejectsUnknownConfigKey` ranges over, but that is a different test). The package is in the card gate's scope, so card 5's tier0 gate fails until the list gains `SamplerSession`.
The focus question on `TestCoverageGuard` itself is settled in the plan's favour: card 5 is the only card that adds a registry row, and it writes `sampler-recipe.yaml` and extends `RecipeEngines` in the same card (embedding it in `recipes.go`, sources list in `internal/loomrecipe/names.go`), so `TestCoverageGuard` sees `SamplerSession` at that gate, and `TestRecipeEngines_ReportsExactlyLoomsOwnEngineSet` is updated alongside. The remaining gap is the name pin above.

### [LOW] Card 6 leaves the AmendmentTarget doc naming four recipes

**Location:** `_lym/plan/06-sampler-recipe.md`; the text is `internal/loomrecipe/names.go`, the `AmendmentTarget` doc line "The recipe names are Rows': shedrun.RecipeTapestry, shedrun.RecipeDarn, shedrun.RecipeBatten and shedrun.RecipePawl, and any other name is refused".
**Issue:** Card 6 maps `shedrun.RecipeSampler` in `embeddedRecipeSource`, so `AmendmentTarget("sampler")` answers instead of being refused, and the doc line becomes false. The card edits the `Rows` and `RowNames` docs (they are listed targets) and `loomrecipe/doc.go`'s sentence on `AmendmentTarget`, but not the function's own doc. The `loomrecipe.RowNames` doc, the `landingshed.Deps` field docs for `CodeReviewOff` and `RerouteReviewed` (card 8 lists `Deps`), the `step_idle_timeout_min` comments (card 12 lists `template.yaml`, `config.go` and `loomCLI`) and `handback.go`'s file comment (card 7 lists the file) all have listed targets; this doc is the one the focus question's checklist misses.

### [LOW] Card 7's doc sweep misses two tapestry-or-darn texts

**Location:** `_lym/plan/07-sampler-vocabulary.md`; the texts are `internal/battencli/wire.go` (`childSeedParams` doc: "tapestry, darn and beam declare one each, params.parent") and `internal/battenshed/deps.go` (`InnerRunDeps.HandBack` doc: "a tapestry or darn escalate halt").
**Issue:** The card widens `childRecipeSupported` to admit sampler, so `childSeedParams` returns `params.parent` for a sampler child (the shared bootstrap in `internal/loomcli/sharedbootstrap.go` writes it for every loom recipe), and the doc's list of recipes that declare one is incomplete. `HandBack` marks a sampler run's fixer escalate halt as the arbiter's too (card 7 itself widens `restepSkips` and `EscalateJudgedFirst` for that), so the field doc is false by the same reasoning as the sibling comments the card does list. Neither function nor struct is a target on any card.

### [MEDIUM] Card 3's stencil would name a path the stencil citation scan refuses

**Location:** `_lym/plan/03-sampler-stencil.md` (Intent, the D9 sentence).
**Issue:** The Intent says the stencil tells the agent to try a dev build "only in another hub on the machine, on lym's own repository the sandbox hub `docs/sandbox-howto.md` describes". `TestStencils_NoBareCrossRepoCitations` (`contracts/stencils/citation_enforcement_test.go`) treats any token under `contracts/`, `docs/` or `internal/` that ends in `.md` or `.go` with no `#` as a bare cross-repository citation; it only passes a `{{.specs_dir}}`-prefixed token or one on `citationAllowlist`. The card plans neither, and the card lists the scan as covering the stencil unchanged. The deployed specs directory does not hold `docs/sandbox-howto.md`, so the `{{.specs_dir}}` form is no fix. A sampler run also works in repositories where the path does not exist.
Test economy in this card is otherwise clean: `TestSessionSpec` asserts only values `SessionSpec` computes (flags, role, segment, skills, the filled directives, record path and slug), the card plans no assertion on the stencil's static prose, and the stencil-wide scans (`TestStencils_NoUnboundedAskOperator`, `TestStencils_NoSkillLoadInstructions`, the directive scans, `TestRegistry_MatchesOnDiskTree`) cover only wording and registration, so none duplicates a `SessionSpec` assertion.

### [LOW] ImpactSummary lines disagree with their cards' Intent

**Location:** `_lym/plan/02-sampler-record.md`, `05-sampler-registry-entry.md`, `06-sampler-recipe.md`, `07-sampler-vocabulary.md`, `08-rows-keyed-wiring.md`, `11-decision-sampler-caller.md` (each card's **ImpactSummary:** line).
**Issue:** One method gap, filed once. Card 2's Intent gives `TestPathConstructors` a row for each of two accessors, the summary says one row. Card 5's summary says the refusal text "changes in one test", but the Intent also changes `checkLoomContract`'s production text, and it omits that widening `LoomBuildEngines` also changes `loomrecipe.verifyGatedRows`, the only other consumer (card 6 narrows it). Card 6's summary omits the `InterruptPolicies` entry. Card 7's summary says no behaviour changes, but the sweep edits agent-facing text (the orch role stencil's two lines, the PR-Review stencil comment). Card 8's summary says each landing seam has one production caller, but `codeBudgetZero` has two (`landingDeps` and `rerouteReviewed`), and it omits `darnDeps` gaining a recipe argument, `wireDarn` losing `budgets`, and the new exported `RowEntryGate` and `CodePerchGate`. Card 11's summary says the change stays inside `decision.go` and its tests, while the Intent also edits `loomcli/doc.go`. The other cards' summaries match their Intent.

## Rejected

- Card 5's `TestSamplerSessionEntry` as a new test beside the `simpleEntryCases` table: the repository keeps per-entry seam tests in per-entry files (`entries_darnwrite_test.go`, `entries_prreview_test.go`), and the card limits the new test to what `TestEveryEntry_RejectsUnknownConfigKey` does not reach.
- `SessionSpec`'s refusal of an empty `StencilsDir`, `WorktreeRoot` or `DecisionRecordPath` with no named test: these are told-value wiring guards, which the refusal rubric exempts, and the card names rows for the two operational refusals (empty slug, unresolvable agent).
- Card 8's claim that `wire` already sets `SegmentBounces` for every recipe, so `wireDarn` can drop its `budgets` parameter: `wire` sets it from `loomrecipe.Routing(c.recipe, loomSegmentBudgets(loomCfg))` in the same form, so the claim holds.
- Card 12's new `loomshed` import of `internal/pawlengine`: neither package imports the other, so no cycle, and the allowlist entry is used by production code, so `AssertImportAllowlistNoStale` passes.

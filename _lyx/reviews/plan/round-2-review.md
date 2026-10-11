---
verdict: APPROVED
findings:
  - id: card4-loomcli-test-budget-row
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/04-split-burler-rows.md
    summary: card 4 adds a top-level test to internal/loomcli, which already sits at its test-budget row, but its Intent raises only the shedrecipe row
  - id: card10-prompt-test-helpers-undecided
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/10-retire-burler-round.md
    summary: card 10 lists testRoundFilePaths, seatPrompts and combinedPrompt as Edit glyphs with no re-sign arrow while its Intent leaves open whether they are deleted, and each one's type or parameters name a type the card deletes
  - id: card2-validatefix-refusals-untested
    severity: MEDIUM
    class: scope
    location: _lyx/plan/02-burler-halves-engine.md
    summary: card 2 prescribes validateFix's refusals but no card names a test that reaches them
  - id: card3-round-complete-ignores-acceptance
    severity: LOW
    class: decision
    location: _lyx/plan/03-burler-halves-producers.md
    summary: D3 says the complete-round check reads the acceptance record, but no card changes roundComplete, which still counts a review and fixer report alone as complete
  - id: card4-env-darn-burler-doc
    severity: LOW
    class: scope
    location: _lyx/plan/04-split-burler-rows.md
    summary: shedrecipe.Env's DarnVerifyAttempts doc names the retired Darn-Burler row, and neither card 4's Env edit nor card 6's prose sweep covers it
---

### [MEDIUM] Card 4 pushes `internal/loomcli` past its test-budget row without saying so

**Location:** `_lyx/plan/04-split-burler-rows.md`, Intent, the `test-budget.yaml` sentence and the tmux smoke test paragraph.

**Issue:** `cmd/lyx/testdata/test-budget.yaml` holds `internal/loomcli: 200`, and the package has exactly 200 top-level tests today, excluding `TestMain`.
Card 4 creates `TestSmokeBurlerReview_AttachesToALiveReviewerInsteadOfRespawning` in `internal/loomcli`, and `TestSmokeBurlerRound_AttachesToALiveRoundInsteadOfRespawning` stays until card 10, so the package reaches 201.
The budget file counts tests "under every build tag", so the tmux tag does not exempt it.
Card 4 lists `test-budget.yaml` under `Edit:`, but its Intent says only that "the `internal/shedrecipe` row of `test-budget.yaml` follows the package's new count".
An implementer following the Intent leaves the `loomcli` row at 200, and card 4's commit fails `TestTestBudget_PackagesStayWithinBudget`.

**Fix:** In card 4's Intent, add that the `internal/loomcli` row of `test-budget.yaml` rises by the new smoke test, beside the `internal/shedrecipe` sentence.

### [MEDIUM] Card 10 leaves the prompt-test helpers' fate open, with no re-sign arrows

**Location:** `_lyx/plan/10-retire-burler-round.md`: the Edit glyphs `internal/burlerengine#testRoundFilePaths`, `#seatPrompts` and `#combinedPrompt`, and the Intent sentence "drop whichever of its helpers no remaining test uses".

**Issue:** `internal/burlerengine/prompt_test.go` declares `var testRoundFilePaths = roundFilePaths{...}`, `func seatPrompts(t, stencilsDir, parentName string, p *Profile, prompts roundPrompts) (reviewer, fixer string)` and `func combinedPrompt(t, stencilsDir string, p *Profile, prompts roundPrompts) string`.
Card 10 deletes `roundFilePaths` and `roundPrompts`, so each of these three helpers is either deleted or re-signed or retyped.
The card lists all three as Edit glyphs, without re-sign arrows, while its Intent leaves the delete-or-keep choice to the implementer.
So the targets cannot match the Intent either way: a deleted helper should be a Delete glyph, and a kept one needs a re-sign arrow, since its parameter type changes.
This is focus item 2's concern, applied to member glyphs that are already listed.

**Fix:** In card 10, decide each of the three helpers.
Move each one no remaining test uses to `**Delete:**` as its member glyph.
Give each kept one a re-sign arrow over card 2's types, for example `internal/burlerengine#combinedPrompt` -> `func combinedPrompt(t *testing.T, stencilsDir string, p *Profile, prompt halfPrompt) string`.
Then replace "drop whichever of its helpers no remaining test uses" with the decided list.
These are member glyphs only, so `redundant-file-target` is not touched.

### [MEDIUM] `validateFix`'s refusals reach no named test

**Location:** `_lyx/plan/02-burler-halves-engine.md`, Intent, the `Engine.Fix` paragraph, and the `fix_test.go` test list.

**Issue:** Card 2 prescribes `validateFix` as a guard with its own refusal set: resolve and check `Target.Paths`, `ReviewPath` and `FixerReportPath`, require a legal `FixScope`, and require non-empty, distinct review and fixer-report paths.
Its inputs arrive from recipe config through `burlerFixProfile` and from `lyx burler run`'s profile, so it is a boundary check.
The test list for `fix_test.go` names no validation case, and card 10 has `TestProfile_Validate` and `TestProfileValidate_ClusterExclude` drive `validateReview` only.
`validateReview` is a different function with a different rule set: it requires fasit and rubric, and `validateFix` deliberately does not.
So no card names a test that reaches any `validateFix` refusal, or that pins a fix profile with no fasit or rubric passing.

**Fix:** Add `internal/burlerengine#TestProfile_Validate` to card 2's `**Edit:**` list, and have the Intent give it `validateFix` rows.
Rows: an empty fixer-report path, identical review and fixer-report paths, an illegal `FixScope` and a missing target path, each refused; and a profile with no fasit or rubric, accepted.
This extends the existing table-driven validation test rather than adding a new one.

### [LOW] The complete-round check does not read the acceptance record, as D3 says it does

**Location:** `_lyx/plan/03-burler-halves-producers.md`, Intent, the acceptance-record paragraph and the `Uses:` entry `internal/shedadapters#highestCompleteRound`; card 10 keeps `roundComplete` unchanged.

**Issue:** D3 states: "round resolution, the complete-round check and archiving read it".
The plan has round resolution read the record (the review row's `highest + 1` branch) and archiving move it, but `roundComplete` (`internal/shedadapters/burler.go:138`) still treats a review and fixer report as complete with no acceptance record.
It feeds `highestCompleteRound` and `hydrationPaths`.
No exit the plan prescribes leaves that pair without its record, so the gap is narrow: a run directory holding such a pair, for example one written before this change, is judged and hydrated as complete instead of being treated as an orphan.
The plan does not record why it departs from D3 here.

**Fix:** Add `internal/shedadapters#roundComplete` to card 3's `**Edit:**` list.
Have it also require the round's acceptance record, and add a row in `burlerreview_test.go` where a review and fixer report without a record are archived and reviewed again.
Alternatively, add a Shared Decision stating that `roundComplete` keeps the review and fixer-report pair because no exit leaves that pair without its record.

### [LOW] `shedrecipe.Env`'s `DarnVerifyAttempts` doc keeps naming `Darn-Burler`

**Location:** `_lyx/plan/04-split-burler-rows.md`, Intent, the `Env` sentence; `_lyx/plan/06-row-name-prose.md`, the `Covered:` list.

**Issue:** `internal/shedrecipe/recipe.go:216`, in `Env`'s `DarnVerifyAttempts` field doc, says loomrecipe stamps the attempts "onto the verify gates of the Darn-Write and Darn-Burler rows".
After card 4 the verify gate rides `Darn-Burler-Fix`.
Card 4's `Env` edit renames only where the field docs "say `BurlerRound`".
Card 6's sweep says it brings every comment naming a retired row up to date, but its enumerated list does not include `shedrecipe#Env`.
So the doc keeps naming a row that no longer exists.

**Fix:** In card 4's `Env` sentence, add that `DarnVerifyAttempts`'s doc names `Darn-Burler-Fix` in place of `Darn-Burler`.
`shedrecipe#Env` is already a card 4 target, so no target changes.

## Focus verdicts

- Segment color on both half tables (focus 1): not a defect.
  Card 2's `reviewTable` carries "the review segment color" and `fixTable` carries `segmentcolor.Review`, matching today's `roundTable` (`internal/burlerengine/engine.go:122`).
  `review_test.go` and `fix_test.go` each pin "the review segment color" in the table's shape, as `PATTERN-spawn-color` requires.
- Delete glyphs for test members removed by Edit cards (focus 2): card 10's `shedadapters`, `shedfake`, `loomcli` and `burlerengine` test members are listed as Delete glyphs, with no bare test-file path.
  Card 4's bare `internal/shedadapters/burler_test.go` and `burler.go` drop only anonymous `var _` assertions, which no member glyph can name, so they are not defects.
  Card 6's bare `overlay_seam_guard_test.go` changes only the file header.
  The one mismatch found is card 10's prompt-test helpers (`card10-prompt-test-helpers-undecided`).

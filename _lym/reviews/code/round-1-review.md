---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/loomrecipe/arrasrecipe_test.go:172
    summary: the arras run-level test drives only the settled re-arm after a Raddle-File return; the goto re-arm and the unsettled-perch goto of D2 have no arras-level case
    fix: in TestArrasRun_SplitPerchReviewsEveryProposalBeforeFiling add a row that enters Raddle-Write by a goto from Raddle-File over a settled perch and asserts a second review round, and a row that enters it by a goto while the perch awaits its parent on an escalation below the hard cap and asserts the Bouncer raises the escalation again with no fresh cycle; leave D4, D5 and D7 to the existing perch-mode, filing and decision tests, which cover them over fixture recipes
  - id: F2
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/loomrecipe/arrasrecipe_test.go:36
    summary: TestArrasRecipe repeats assertions that existing tests already pin (row order, HoldsSplitPerch, Discussion approval row)
    fix: delete the arrasRowNames slice with its slices.Equal assertion (TestRowNames_FollowEachRecipesRowOrder and TestEntryNumbers pin the arras rows), the HoldsSplitPerch(arras) assertion (TestHoldsSplitPerch) and the DiscussionApprovalRow(arras) assertion (TestDiscussionApprovalRowsCarryApproveSeam); keep the Motif SkipTo, routing, segment, absent-seam and fixer-gate assertions
  - id: F3
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/loomengine/review.go:154
    summary: new split_* config behaviors have no row in the existing table tests, which are the places to cover them
    fix: add rows to the existing tests, no new test functions - a non-empty split_judge in the ResolveJudge test, an empty split_fixer_model with raddle empty (keeps fixer_model) in the ResolveReview test, a negative split_max_bounces in the LoadConfig budget refusal test, Raddle-Bouncer to split in TestBouncerRunSubdir, and an assertion that each arras block of runprofile/template.yaml sets the split_* and discussion_* keys the ladder step names
  - id: F4
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: docs/overview.md:1522
    summary: the batten recipe lists still say tapestry, motif, raddle, darn and sampler, though childRecipeSupported now admits arras
    fix: add arras to the recipe list at docs/overview.md:1522 with a one-line description ("arras is for a child that discusses a task and splits its decision record into board entries") and to the Run-Shed list at docs/overview.md:1791
  - id: F5
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: docs/overview.md:1322
    summary: the loom.yaml key paragraphs describe only the discussion and code perch keys, not the new split_* keys
    fix: in the loom.yaml paragraphs at docs/overview.md:1318-1347 and 1493-1499 add the split perch's keys - split_max_bounces, split_hard_max_bounces and split_auto_accept beside the discussion and code ones, split_reviewer_model, split_fixer_model (an empty value takes the raddle key's model) and split_judge, and split_reviewer_fan and split_fixer_fan (closed to the local layer)
---

### [LOW] The arras run-level test covers one re-arm path

**Location:** internal/loomrecipe/arrasrecipe_test.go:172
**Issue:** D2 states three entries into Raddle-Write: a Raddle-File return, a goto from Raddle-File, and a goto from a perch that has not settled.
The diff changes no engine code for them, so they rest on the Bouncer's existing clear.
The new test pins only the first, and a regression in the other two would pass.
The acceptance criteria for D4, D5 and D7 are covered over fixture recipes by existing tests plus the arras-specific detection rows (`HoldsSplitPerch`, `DiscussionApprovalRow`), so they are not a gap of this diff.

### [LOW] TestArrasRecipe duplicates existing coverage

**Location:** internal/loomrecipe/arrasrecipe_test.go:36
**Issue:** The row-order check repeats `TestRowNames_FollowEachRecipesRowOrder`'s arras row and `TestEntryNumbers`' arras row.
The `HoldsSplitPerch(arras)` check repeats `TestHoldsSplitPerch`, and the `DiscussionApprovalRow(arras)` check repeats `TestDiscussionApprovalRowsCarryApproveSeam`.
Each is a redundant assertion of a behavior another test covers.

### [LOW] New split_* config behaviors lack table rows

**Location:** internal/loomengine/review.go:154
**Issue:** `split_judge` resolution, the empty-raddle branch of the split fixer fallback, the `PerchSplit` entries of the budget bounds, the `split` run subdirectory of `BouncerRunSubdir` and the `arras` blocks of the run-profile template are each pinned only by their default value or not at all.
The change description claims a bare `Config` keeps `fixer_model`; no test shows it.

### [LOW] Batten recipe lists in docs/overview.md omit arras

**Location:** docs/overview.md:1522
**Issue:** The batten paragraph says the child recipe is one of five names, and the `Run-Shed` paragraph at line 1791 lists the same five.
`internal/battencli/wire.go:607` now admits `arras` too, and the same diff updated three other recipe lists in this file.

### [LOW] docs/overview.md does not describe the split_* keys

**Location:** docs/overview.md:1322
**Issue:** The loom.yaml key paragraphs name the discussion and code perch keys for budgets, auto-accept, models, fans and judges.
The diff adds eight `split_*` keys to `loom.yaml`, one of them a guard key and one with a model fallback to `raddle`, and none appears there.

## Rejected

- Cold proposal is stored before the split fixer's edits, so `lym report raddle` reads the fixer's changes as operator edits (code-correctness F1): no decision in the record asks for the report to treat arras differently, the store-once rule is pre-existing, and a change would belong to the report or raddle-state code the Scope packages do not list.
- Possible nonexistent `split_seed` key named in a bad-seed error (code-correctness F2): `resolveSegmentBouncerModels` names the key `seed` or the judge key for an empty segment seed, never `split_seed`, so the worry does not hold.
- Boardengine and battencli branches that stay raddle-only (code-affected-set note): the filing of an arras entry belongs to the record's later decisions, outside D1-D3.
- Run-level coverage for D4, D5 and D7 as a MEDIUM gap (code-tests F1, the part beyond the goto cases): the diff adds no behavior there, the existing perch-mode, filing and decision tests cover them, and the arras-specific ties are pinned by `HoldsSplitPerch` and `DiscussionApprovalRow`; severity lowered and narrowed to the D2 goto cases in F1.

## Focus verdicts

- Recipe rows, `on_done` order, the Motif `undiscussed` gate, Raddle-File's `on_stuck` return, no `Record-Write` row, and the split perch rows (artifact, rubric, `fix-scope: overlay`, `raddle` gate, no `carry_over`, `approve_seam` or `commit_seam`, `escalate_decisions`, `decisions_escalated`) match D1 and D2.
- The re-arm needs no new code: the Bouncer clears a settled verdict at entry when it has no Subject seam, and the settled path is pinned by the second case of the run-level test; the unsettled-goto path is the gap in F1.
- All eight `split_*` keys are present; an empty `split_fixer_model` takes the raddle key's model; `split_auto_accept` defaults to false and is a guard key through `reviewbudget.Categories`.
- Registration sites for `arras` are complete for every place `motif` and `raddle` are registered as recipes; the former-name case is reversed, not deleted.
- The rubric names the decision record and its `Added after Discussion` entries, each D3 judgement, and leaves the mechanical checks to the gate; it is embedded and registered with the other loom rubrics.
- The diff adds no refusal and branches on no recipe name; the perch-mode `raddle-approve` refusal it relies on names a way forward.
- The commit bodies' claims hold in the diff; the run-level tests fake Raddle-Write and Raddle-File, as the change description states.

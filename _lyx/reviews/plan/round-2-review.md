---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    location: _lyx/plan/17-single-llm-delete.md
    summary: Card 17 rewords the comments citing the single producer but misses three Env field comments, a shed-recipe-spec bullet and the buildSequenceFixture doc comment, and card 20's grep cannot catch them
    fix: In card 17's Intent, extend the comment-reword bullet to name the doc comments of the Env fields AnchorPath, WorktreeRoot and StencilsDir in internal/shedrecipe/recipe.go, which say they are read by SingleLLM or SingleLLM's tokens and should name the readers that remain. Also name the Config bullet of contracts/specs/shed-recipe-spec.md, whose example engine is SingleLLMProducer and should become MultiLLMProducer. Both are already card 17 targets (internal/shedrecipe#Env and //contracts/specs/shed-recipe-spec.md), so no target changes. In card 14's Intent, the last card to edit internal/loomrecipe#buildSequenceFixture, add that its doc comment describes the Discussion-Write, Plan-Write and Describe rows as running on the seat producer over env.Seats, in place of its SingleLLMProducer, env.Shuttle and env.DiscussionSpec text.
  - id: F2
    severity: LOW
    class: consistency
    location: _lyx/plan/05-checklist-review-rounds.md
    summary: Card 5 adds goto entries to the history-order list that card 4's unreadable-recipe frame walks, yet says that frame stands unchanged, so whether a goto line shows there is left open
    fix: In card 5's Intent, replace the sentence about the unreadable recipe with one that says the unreadable-recipe frame skips the goto lines card 5 adds to the history-order list, so it still shows only the visit lines D1 names. Alternatively, say it draws them as goto lines and add that case to card 5's TestRenderChecklist rows.
  - id: F3
    severity: NIT
    class: consistency
    location: _lyx/plan/08-seat-lead-naming.md
    summary: Card 8 says no code reads RoleChair, RoleAdvisor or AdvisorName after it, while card 20 says DiscussionTable and its test name them until card 19
    fix: In card 8's Intent, say that DiscussionTable forms its placeholder advisor names with seatengine.AdvisorName, its last production reader until card 19, and keep card 20's ordering rationale as written. Alternatively, have card 8 form advisor-<n> without AdvisorName and drop DiscussionTable from card 20's rationale.
---

### [MEDIUM] Card 17 misses single-producer comment sites

**Location:** `_lyx/plan/17-single-llm-delete.md`

**Issue:** `internal/shedrecipe/recipe.go` names the deleted row in three `Env` field doc comments.
`AnchorPath` is "read by Batchifier, Webster, Bouncer, SingleLLM's anchor_path token", `WorktreeRoot` is "read by SingleLLM's output_files" and `StencilsDir` is "read by SingleLLM and Bouncer".
Card 17 lists `internal/shedrecipe#Env` as a target, but its Intent rewords only the `Shuttle` field's comment.
`contracts/specs/shed-recipe-spec.md`'s `Config` bullet says "This is what lets one `Engine` (e.g. `SingleLLMProducer`) serve many rows".
That is an example, not the adapter listing that card 17 says it removes.
`internal/loomrecipe/fixture_test.go`'s `buildSequenceFixture` doc comment describes rows 3 and 6 as "a real shedadapters.SingleLLMProducer" over `env.Shuttle` and `env.DiscussionSpec`.
Cards 11, 12 and 14 move those rows onto `env.Seats` but never say the comment changes, and card 17's test-comment list does not name it.
Card 17's ImpactSummary claims that "the remaining references are comments listed here", which these sites contradict.
Card 20's Verify grep matches `SingleLLMProducer` only, never `SingleLLM's` or `SingleLLM and`, and it skips `.md` and test files, so none of these would be caught.
The repo rule that docs land in the same commit, and the decision record's `PATTERN-documentation-lifecycle` constraint, require the rewording.

### [LOW] Card 5 leaves goto lines in the unreadable-recipe frame undecided

**Location:** `_lyx/plan/05-checklist-review-rounds.md`

**Issue:** Card 4 builds the unreadable-recipe frame by walking `checklistVisitsOf`'s history-order result.
Card 5 then "records a goto entry in that list as a zero-time line (outcome `goto`)" and also says "With the recipe unreadable, card 4's history-order list stands unchanged".
After card 5, the list holds goto lines.
Card 5 says card 4's per-row lines keep leaving gotos out, but it says nothing of the unreadable walk, so that frame either gains goto lines or must filter them.
D1 says that frame shows "every visit line in history order", and card 4 already counts a goto as no visit.
Today `checklistVisitsOf` (`internal/loomcli/checklist.go`) skips goto entries entirely.

### [NIT] Cards 8 and 20 disagree on when AdvisorName loses its last reader

**Location:** `_lyx/plan/08-seat-lead-naming.md`

**Issue:** Card 8 says "No code reads `RoleChair`, `RoleAdvisor` or `AdvisorName` any more", yet it keeps `DiscussionTable` naming advisors `advisor-<n>`.
Today `internal/loomengine/discussiontable.go:101` does that through `seatengine.AdvisorName(n + 1)`.
Card 20 says card 19 is "the last card that edits code naming them (`DiscussionTable` and its test)".
Either order builds, but the two cards give the implementer conflicting statements.

## Focus verdicts

- Card 4 against card 5: card 4 relies on nothing that card 5 adds.
  Card 4 itself re-signs `checklistVisitsOf` with the third, history-order result, and adds the `row` field that the unreadable-recipe list names.
  Card 5 adds only `index`, `isGoto` and goto lines, which card 4's frame does not read (see F2 for the goto side effect).
  Not a defect.
- The unparsable twill `burler.yaml` refusal stays reachable.
  Card 11's `TestDiscussionTable_Refuses` and card 12's `TestPlanTable` each gain a row whose `burler.yaml` does not parse.
  Card 19 keeps those rows with the lens key set.
  `burlerengine.LoadConfig` decodes strictly and returns the parse error that `twillLenses` wraps, so no upstream check intercepts it.
  An empty fan likewise reaches `burlerengine.ResolveFan`'s "is empty" error, not a load error.
  Not a defect.
- Doc coverage for card 17: two package `doc.go` files mention the single-LLM constructor, `internal/shedadapters/doc.go` and `internal/shedbuild/doc.go`, and card 17 covers both.
  The `doc.go` files of `landingshed` and `darnengine` name the retired spec builders and are covered by card 18.
  The member-glyph check passes for every Edit card, since none lists only Go file paths while naming symbols.
  Card 17's file targets `internal/shedadapters/gateattempts.go`, `archive.go` and `internal/mergeresolve/spec.go` carry file-header edits.
  The gaps that remain are in non-`doc.go` comments (F1).

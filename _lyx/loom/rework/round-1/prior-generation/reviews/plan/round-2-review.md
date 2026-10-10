---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/plan/07-discussion-seats-entry.md:42
    summary: Card 7 adds the func-typed Env.DiscussionTable but only card 8 fills it in loomcli wire, so loomcli's TestWire_EverySeamFilled is red from card 7 until card 8
  - id: F2
    severity: BLOCKING
    class: scope
    location: _lyx/plan/08-discussion-write-row-selection.md:43
    summary: Card 8 makes RecipeEngines return DiscussionSeats, but TestRecipeEngines_ReportsExactlyLoomsOwnEngineSet pins RecipeEngines to wantProducerTable's engine set and is not targeted, so it fails
  - id: F3
    severity: MEDIUM
    class: scope
    location: _lyx/plan/00-overview.md:27
    summary: No card carries the acceptance criterion that the discussion gate and the parent-review brief read only the two discussion files with an advisor notes file beside them
  - id: F4
    severity: LOW
    class: design
    location: _lyx/plan/04-discussion-chair-and-advisor-stencils.md:24
    summary: The chair prompt opens with the included discussion stencil's claim that it is a single agent, and that stencil's header still names composePrompt and FillOptional as its only filler; card 4 addresses neither
  - id: F5
    severity: LOW
    class: consistency
    location: _lyx/plan/08-discussion-write-row-selection.md:45
    summary: Card 8's DiscussionSeats-false subtest re-asserts what TestNew_ShapeMatchesRecipe and the existing sequence tests already cover
---

### [BLOCKING] Card 7 leaves loomcli's seam-completeness test red until card 8

**Location:** `_lyx/plan/07-discussion-seats-entry.md:42` (card 7's `intentionallyNil` line), with `_lyx/plan/08-discussion-write-row-selection.md:46` (card 8's `wire` fill).

**Issue:** Card 7 adds `Env.DiscussionTable`, a `shedadapters.TableSource`, which is a func type.
`envkit.NilSeams` (`internal/testkit/envkit/envkit.go:244-272`) reports every nil `reflect.Func` field, and `internal/loomcli/wiring_seams_test.go:47-66` (`TestWire_EverySeamFilled`) fails for any nil path that `loomcli`'s own `intentionallyNil` does not list.
Card 7 adds the field to `internal/battencli#intentionallyNil` only.
`loomcli`'s `wire` fills `DiscussionTable` only in card 8, so from card 7 through card 8 `lyx gate test ./internal/loomcli` is red.
This breaks the "Plan vs. schedule" rule: card 7's result depends on work in a later card.
It also contradicts the overview's own `coverage-guard-bridge` rationale that "each card builds and tests green on its own".

**Fix:** Move the `DiscussionTable` fill in `internal/loomcli#loomCLI.wire` into card 7, which may call `loomengine.DiscussionTable` because card 5 comes first; card 8 then wires only `DiscussionSeats`.
Alternatively, add a bridge entry for `DiscussionTable` to `internal/loomcli`'s `intentionallyNil` in card 7, and have card 8 remove it, which the test's reverse check forces, as the coverage-guard bridge does.

### [BLOCKING] Card 8 breaks `TestRecipeEngines_ReportsExactlyLoomsOwnEngineSet`

**Location:** `_lyx/plan/08-discussion-write-row-selection.md:43`.

**Issue:** Card 8 makes `RecipeEngines` return the recipe's engines plus `discussionSeatsEngine`.
`internal/loomrecipe/recipe_test.go:206-230` asserts that `RecipeEngines()` equals, by length and element, the deduplicated engine column of `wantProducerTable`.
Card 8's own second subtest keeps `wantProducerTable` describing the default build, so that column still lacks `DiscussionSeats`.
Neither the test nor `wantProducerTable` is in card 8's `Edit:` list, so `RecipeEngines()` returns one engine more than the test expects and the test fails.
The plan's `## verify:` `go test ./...` would fail.

**Fix:** Add `internal/loomrecipe#TestRecipeEngines_ReportsExactlyLoomsOwnEngineSet` to card 8's `Edit:` list, and say in the Intent that its want set is `wantProducerTable`'s engines plus `discussionSeatsEngine`, so the guard still catches a silently empty or shrunk return.

### [MEDIUM] No card carries the gate-and-brief acceptance criterion

**Location:** `_lyx/plan/00-overview.md:27` (Shared Decisions; no card carries the criterion).

**Issue:** The decision record's acceptance criteria include "The discussion gate and the parent-review brief read the two discussion files only, with an advisor notes file present beside them", which is D7's bound.
The parent's supersession keeps D7: the seat-table producer still writes `advisor-<n>.md` into `_lyx/discussion/`.
No card pins it.
Card 8's loomrecipe test uses a table of the chair alone, so no notes file exists beside the files the real discussion gate judges.
Its parent-review config stubs `RenderBrief`.
No other card mentions the discussion gate or the brief.

**Fix:** Add a row to `internal/loomshed#TestNewDiscussionGate` that plants an `advisor-1.md`, whose content would fail the gate if read, beside a valid decision record and support log, and asserts the gate result is unchanged.
Add a row to `internal/loomengine#TestParentReviewBrief_NamesPathsAndSubmitLines` asserting that the brief names only the two discussion files while the notes file is present.
Either fits an existing card, such as card 5 for the brief row or card 8 for the gate row, or a small card after card 5.

### [LOW] The included discussion stencil's text still describes the single-agent path

**Location:** `_lyx/plan/04-discussion-chair-and-advisor-stencils.md:24`.

**Issue:** The chair stencil renders `{{template "loom-template-discussion"}}` first.
So the chair's prompt opens with that stencil's "You are the Discussion producer: a single agent running the one interactive phase of a loom task" (`contracts/stencils/loom/loom-template-discussion.md:14`), and only after the whole stencil, Step 6 and the fence included, comes the chair directive and the advisors section.
Card 4's advisors section and claims never say that the chair directive supersedes the single-agent sentence.
The discussion stencil's header comment (lines 1-11) also still names `composePrompt` and `stencil.FillOptional` as its only filler, which becomes false once the seat engine fills it through `FillWith` as an include.
The `chair-stencil-includes-the-discussion-stencil` decision freezes the stencil's bytes to keep the single-agent prompt unchanged.
The leading comment is stripped before every fill (`internal/stencil/stencil.go:36`, `IncludeNames`), so editing it changes no prompt.

**Fix:** In card 4, have the advisors section open by stating that the chair works with the advisors named above, which replaces the included single-agent framing, and pin that sentence with a claim.
Also edit the discussion stencil's leading header comment to name the chair stencil's include as its second filler, narrowing the bytes-unchanged decision to its agent-facing body.

### [LOW] Card 8's false-branch subtest repeats existing coverage

**Location:** `_lyx/plan/08-discussion-write-row-selection.md:45`.

**Issue:** The second subtest of `TestNew_DiscussionSeatsRunsTheWriterRowOnSeats` asserts that `wantProducerTable` still describes the built list with `DiscussionSeats` false.
`TestNew_ShapeMatchesRecipe` (`internal/loomrecipe/recipe_test.go:22`) already asserts exactly that, for the default env.
The subtest also asserts that the shuttle ran the `discussion` role, which the existing sequence tests (for example `TestResume_DiscussionWriteRespawnsRatherThanReportDoneOffFileExistence`) already drive.
Only "the seat fake saw no table" is new.

**Fix:** Cut the subtest down to the one new assertion, that the seat fake saw no table when `DiscussionSeats` is false, or fold that assertion into `TestNew_ShapeMatchesRecipe` and drop the subtest.

## Focus verdicts

- Discussion-stencil card order: no card adds `{{template "seat-directive-chair"}}` to `loom-template-discussion.md`, and no card deletes or replaces `loomengine.composePrompt`.
  Card 4 creates `loom-template-discussion-chair`, which includes the unchanged discussion stencil.
  `stencil.FillWith` and `seatengine.validateStencil` accept that, because the discussion stencil declares no include and its leading comment, which spells `{{.X}}`, is stripped before parsing.
  The single-agent composer keeps filling the unchanged stencil, so no card between them leaves a test or the live Discussion-Write row unable to compose.
  Not a defect.
- `shedfake.SeatRunner` change: card 6 is a card of its own, placed before card 8, its only consumer.
  Among the fake's consumers in `internal/shedadapters/multillm_test.go`, `internal/shedrecipe/entries_multillm_test.go`, `internal/shedrecipe/multillm_fixture_test.go` (whose `testdata/multillm-recipe.yaml` carries no `gates`) and `envkit.FullEnv`, only `TestMultiLLMEntry_ComposedTable` scripts a gated table with a done chair and no `GateOutcome`, and card 6 targets it.
  Not a defect.
- Double gate evaluation: card 6's Bound states that a `RunFn` or `ResumeFn` returning its own `GateOutcome` wins and the fake then never evaluates the gate.
  Card 8's `newLoomSeats` returns no `GateOutcome`, so the fake evaluates the gate exactly once.
  Not a defect.
- Decision-record fidelity: the Shared Decisions follow from the parent's decision superseding D1.
  Keeping the `discussion` role key, adding a separate chair stencil and dropping the parent-name parameter are each recorded with rationale.
  `absent-key-takes-template` repeats the record's own amended reading of D4: an absent key takes the template's one advisor, and "absent loads as zero advisors" holds for an in-memory `Config`.
  `resolve-at-load-and-in-the-builder` meets D4's load-time parse-and-resolve refusal.
  Validating the list whatever `discussion_producer` holds is a recorded choice and does not break the record's "read only by the seat-table producer", which concerns runtime use.
  The one fidelity gap found is F3.

## Prior rounds

I read round 1's review and fixer report after saving the findings above.
Every round 1 finding is fixed in the current plan, and none has regressed: the chair include sits in a new stencil, the fake's gate-once change has its own card that states which evaluator wins, the mapping refusal carries the advisor way forward, the seat-engine and producer changes are additive, and the reconcile coverage is rows in existing tests.
The stale-`DiscussionSpec` comments no longer arise, because the single-agent path stays.
The deferred absent-key half of `d4-deviations-unrecorded` is now settled by the decision record's own parent entry superseding D1, which the plan's `absent-key-takes-template` follows.

---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/plan/04-checklist-combined-frame.md:33
    summary: Card 4 prescribes the unreadable-recipe list in history order across rows, but the per-visit history-order record that list needs is added only by card 5, so card 4 depends on a later card
    fix: In card 4's Intent, under the Outcome words paragraph, state that checklistVisitsOf also returns every visit in history order, each carrying its new row field, as a third result beside the per-row map and the first-visit row order, and that the unreadable-recipe list walks that result. Give card 4's checklistVisitsOf Edit bullet the arrow shown in F2. In card 5's Intent, reword the checklistVisitsOf sentence so that card 5 adds only the index and isGoto fields and the goto lines to the history-order list card 4 already returns.
  - id: F2
    severity: LOW
    class: consistency
    location: _lyx/plan/07-checklist-pause-net-time.md:10
    summary: Edit bullets lack the re-sign arrow where the Intent changes parameters or results, namely checklistVisitsOf in cards 4 and 7 and the stuckEntry test helper in card 4
    fix: "In 04-checklist-combined-frame.md, replace the bullet `internal/loomcli#checklistVisitsOf` with `internal/loomcli#checklistVisitsOf` -> `func checklistVisitsOf(status shedengine.Status, startedAt string, now time.Time) (visits map[string]checklistVisits, order []string, history []checklistVisit)`, and the bullet `internal/loomcli#stuckEntry` with `internal/loomcli#stuckEntry` -> `func stuckEntry(producer string, atMinute int, routedTo string) shedengine.HistoryEntry`. In 07-checklist-pause-net-time.md, replace the bullet `internal/loomcli#checklistVisitsOf` with `internal/loomcli#checklistVisitsOf` -> `func checklistVisitsOf(status shedengine.Status, startedAt string, paused []pauserecord.Interval, now time.Time) (visits map[string]checklistVisits, order []string, history []checklistVisit)`. Each overlay passes validate-plan --overlay on its own and together."
  - id: F3
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/19-loom-config-axes.md:28
    summary: Card 19 has twillLenses, discussionTwillDirective and planTwillDirective take the fan name and its key, but their Edit bullets carry no re-sign arrow, and the gate refuses the arrow fix
    fix: "No admissible fix: with the arrows added to card 19, validate-plan --overlay reports caller-uncovered (blocking) on card 19, because planTwillDirective is called at internal/loomengine/plan.go:142 inside PlanSpec and discussionTwillDirective at internal/loomengine/discussion.go:97 inside DiscussionSpec, which card 18 deletes and so no card 19 target covers. The fixer either finds a placement the caller-uncovered check admits, or defers this finding naming caller-uncovered as the constraining check."
  - id: F4
    severity: MEDIUM
    class: scope
    location: _lyx/plan/19-loom-config-axes.md:70
    summary: The burler.yaml-unreadable twill refusal loses its only tests, the does-not-parse rows of the TestDiscussionSpec_Refuses and TestPlanSpec_Refuses tests that cards 11 and 12 delete, and card 19 keeps that refusal with no named test
    fix: In card 11's Tests, have TestDiscussionTable_Refuses gain a twill-discussion row whose burler.yaml does not parse (for example fans [unterminated), expecting twill-discussion and burler.yaml in the error. In card 12's Tests, have TestPlanTable's refusals include the same row for twill-plan. In card 19's Tests, have both rows set the lens key and expect the key's name in the error, beside the missing-fan and empty-fan rows.
  - id: F5
    severity: MEDIUM
    class: scope
    location: internal/shedbuild/doc.go:11
    summary: shedbuild's package doc says the single-LLM constructor probes its stencil at build time, and card 17 deletes that constructor and edits shedbuild's tests without editing the doc
    fix: Add `internal/shedbuild/doc.go` to card 17's Edit list, and add to its Intent that shedbuild's package doc drops the single-LLM constructor from its list of constructors that reach disk at construction time. This overlay passes validate-plan --overlay.
  - id: F6
    severity: LOW
    class: consistency
    location: _lyx/plan/17-single-llm-delete.md:45
    summary: Card 17 lists two test-file paths for comments that are doc comments of named existing tests, where the plan template asks for member glyphs
    fix: "In card 17's Edit list, replace `internal/loomcli/smoke_gate_test.go` with `internal/loomcli#TestSmokeGate_RepromptsThroughARealPaneAndFixesTheArtifact`, and `internal/shuttleengine/attach_test.go` with `internal/shuttleengine#TestAttach_RunningRecordSatisfiedFileContract_HarvestsNotRespawn`, then reword the Intent's test-file bullet to name the two tests. Keep `internal/loomshed/planwrite_test.go`, whose comment is the file header. This overlay passes validate-plan --overlay."
---

### [MEDIUM] Card 4's history-order list needs a record only card 5 adds

**Location:** `_lyx/plan/04-checklist-combined-frame.md:33`, `:37`; `_lyx/plan/05-checklist-review-rounds.md:22`

**Issue:** D1 asks for the unreadable-recipe frame to show every visit line in history order, each naming its row, and card 4 plans that list and its test row.
Today `checklistVisitsOf` (`internal/loomcli/checklist.go:259`) returns only a per-row map of visit logs and the rows in first-visit order, so the cross-row order is lost.
Card 4 adds only a `row` field to each visit.
Card 5 then adds what the list needs: "every visit in history order with its row".
So either card 4 cannot meet its own Intent and test, or its implementer builds card 5's record ahead of time, and card 5 then prescribes work that already exists.
This breaks the plan-vs-schedule rule: card 4 depends on what card 5 does.

### [LOW] Missing re-sign arrows on the checklist's visit collector and test helper

**Location:** `_lyx/plan/07-checklist-pause-net-time.md:10`; `_lyx/plan/04-checklist-combined-frame.md:12`, `:13`

**Issue:** Card 7's Intent says `checklistVisitsOf` "takes" the paused intervals, which is a new parameter, and its bullet has no arrow.
Card 4's Intent says `stuckEntry` "gains a routed-to argument" (its current head is `func stuckEntry(producer string, atMinute int) shedengine.HistoryEntry`, `internal/loomcli/checklist_test.go:33`), and its bullet has no arrow either.
Once F1 moves the history-order result into card 4, card 4's `checklistVisitsOf` bullet also changes its results.

### [MEDIUM] Card 19's twill functions are re-signed without arrows, and the gate refuses the arrows

**Location:** `_lyx/plan/19-loom-config-axes.md:28`, `:29`, `:30`, `:70`

**Issue:** Card 19's Intent says "`twillLenses`, `discussionTwillDirective` and `planTwillDirective` take the fan name and its key".
Today they take a fan name hard-wired by the constants card 19 deletes (`internal/loomengine/twill.go:39`, `:57`, `:81`), so their parameters change, yet none of the three bullets carries the arrow.
I overlaid the arrows for card 19 alone.
`lyx loom validate-plan --overlay` then reports `caller-uncovered` (blocking): the two directives are still called from `PlanSpec` (`internal/loomengine/plan.go:142`) and `DiscussionSpec` (`internal/loomengine/discussion.go:97`), which card 18 deletes before card 19 runs.
The baseline `{}` run has no blocking finding, so I withheld that fix and name the constraining check.

### [MEDIUM] The burler.yaml-unreadable twill refusal ends up untested

**Location:** `_lyx/plan/19-loom-config-axes.md:70`, `:89`; `_lyx/plan/11-discussion-write-seats.md:95`; `_lyx/plan/12-plan-write-seats.md:50`

**Issue:** `refusal-spec.md` holds the rows "Discussion-Write: burler.yaml unreadable for twill" and "Plan-Write: burler.yaml unreadable for twill".
Only the "twill burler.yaml does not parse" rows reach them: one in `TestDiscussionSpec_Refuses` (`internal/loomengine/discussion_test.go:113`) and one in `TestPlanSpec_Refuses` (`internal/loomengine/plan_test.go:226`).
Card 11 deletes the first test, and its `TestDiscussionTable_Refuses` gains only the unresolvable-fan row.
Card 12 deletes the second, and `TestPlanTable` lists only the unparsable model-spec, missing-stencil and unresolvable-fan refusals.
Card 19 then keeps this refusal ("an unreadable `burler.yaml` is refused naming the key") and rewords its spec rows, but its tests cover only the missing fan and the fan with no lenses.
That leaves a prescribed refusal with no named test that reaches it.
It is a correctness halt with its own row, not a member of a grouped transient row.

### [MEDIUM] shedbuild's package doc keeps describing the deleted single-LLM constructor

**Location:** `internal/shedbuild/doc.go:11`; `_lyx/plan/17-single-llm-delete.md:43`

**Issue:** `shedbuild/doc.go` says that three registry constructors reach disk at construction time, among them that "the single-LLM constructor only probes its stencil".
Card 17 deletes `entries_singlellm.go` and the `SingleLLM` row, and edits shedbuild's `engineMinimalConfig` and `TestBuild_Errors`, but not this doc.
The decision record's documentation constraint says every touched package's `doc.go` changes with the code.
Card 20's Verify grep cannot catch this, because the doc spells "single-LLM", not `SingleLLMProducer`.

### [LOW] Card 17 names two tests by their file paths

**Location:** `_lyx/plan/17-single-llm-delete.md:45`, `:46`

**Issue:** The comments card 17 rewords in `smoke_gate_test.go:133` and `attach_test.go:600` are the doc comments of `TestSmokeGate_RepromptsThroughARealPaneAndFixesTheArtifact` and `TestAttach_RunningRecordSatisfiedFileContract_HarvestsNotRespawn`.
The plan template keeps a plain test-file path only for a file the card adds tests to without changing an existing member.

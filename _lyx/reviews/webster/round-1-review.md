---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: scope
    location: internal/treadleengine/targeting.go:37
    summary: runTargeting's doc comment cites the composePrompt convention of loomengine, a function card 18 deleted
    fix: In the doc comment of runTargeting, drop the clause "(see the composePrompt convention this mirrors)" and end the sentence after "still compiles", so no comment names the deleted builder.
  - id: F2
    severity: LOW
    class: scope
    location: cmd/lyx/pathrules_test.go:54
    summary: The pathRules comment names TestPlanSpec_PatternDirective in internal/loomengine/plan_test.go, a test card 12 deleted
    fix: In the pathRules comment, replace TestPlanSpec_PatternDirective with the test that now proves the pattern.File call site's root, the "PATTERN.md is read from the worktree root" subtest of TestPlanTable in internal/loomengine/plan_test.go.
---

### [LOW] runTargeting cites a deleted function

**Location:** `internal/treadleengine/targeting.go:37`
**Issue:** Card 18 deletes `loomengine.composePrompt` and its siblings, and the focus asks that no comment still cite a deleted symbol.
This comment is the one site the card's edit list does not name and the grep for the deleted names still finds outside `seatengine`.
The only `composePrompt` left is the unexported one in `internal/seatengine/prompt.go`, which is not the convention the comment means (a leading `stencilsDir` argument), so the pointer is now false.

### [LOW] pathRules comment names a deleted test

**Location:** `cmd/lyx/pathrules_test.go:54`
**Issue:** Card 12 deletes `TestPlanSpec_PatternDirective`, and the diff already rewrites the line above this one (`PlanSpec` to `PlanTable`) in this same comment block, so the sentence two lines later was missed.
`internal/loomengine/plan_test.go` now holds `TestPlanTable` alone, and the PATTERN.md-at-worktree-root case lives in its `"PATTERN.md is read from the worktree root"` subtest.

## Checked and not a defect

- **Cards 16, 17, 18, 20.**
  `singlellm.go` to `shuttle.go` is a pure `git mv` in `374efea4f` (0 changed lines), and `writeStencilFile` moved unchanged.
  Card 20's own Verify grep (`SingleLLMProducer`, `RoleChair`, `AdvisorName` in non-test `.go` files under `internal cmd contracts tools`) returns nothing.
  No Go file outside tests names `DiscussionSpec`, `PlanSpec`, `ReworkSpec`, `DescribeSpec`, `DarnSpec`, `DiscussionSeats`, `SingleLLM` or `entries_singlellm`.
  The remaining hits are test names kept by card 18 (`TestDescribeSpec_*`, `TestDarnSpec*`) and `docs/benchmarks` reports, which record past runs.
- **Card 19.**
  `MigrateConfig` maps `twill` to `discussion_lenses: twill-discussion` and `plan_lenses: twill-plan` and every other value to `""`.
  It sets `discussion_advisors` to `[]` for any non-`seats` value even when absent, so a lower layer's list stays off.
  It keeps string entries only under `seats`, as `advisor-<n>` with a report line, leaves a `seats` document with no list of its own alone with a report line, and returns a document without a retired key unchanged.
  The old twill and seats forks therefore keep their advisor and lens behavior.
  The retired keys survive only inside the migration; `grep` finds `discussion_producer` and `plan_producer` in `config.go`, `config_test.go` and the migration paragraph of `docs/overview.md` alone.
- **Cards 9, 10 and 11 to 15.**
  The amendment rides a copy of the table (`freshTable`) on the fresh path only, so the probe and resume tables carry none, and `amendLeadPrompt` appends it to the lead alone.
  `mapOutcome` now carries the start error text on a not-started lead.
  The converted rows keep their `gates` stamping, their fresh-spawn preparations and their `requireSeam` refusals, now naming the table and `Seats`.
  The Discussion lead of a hub that had no advisors now opens with the chair directive block; card 11 and the `loom-template-discussion-chair` header set that, and the directive is worded for a lone lead.
- **Card 8.**
  `Table.Validate` pins the empty name to `Seats[0]`, refuses a later seat that is unnamed, invalid or duplicated, and refuses a multi-seat lead without `seat-directive-chair`.
  Every reader of a seat's role goes through `SeatRole`, which returns the prefix alone for the lead; `prompt.go`, `engine.go` and `resume.go` compare against `leadName` and key the lead under `""`.
  `multiLLMSeat` refuses a named first seat and requires a name on every later one.
- **Cards 1 to 7.**
  `routed_to` is written only on the OnStuck bounce arm and is not read by `episodeStuckCount`, which looks at `OutcomeReroute` entries alone.
  A fold now also needs equal `RoutedTo`.
  The checklist reads `bounced` only for a Stuck entry with a `routed_to`, so an older binary's entry reads as stored.
  Pause coverage is clipped and de-overlapped by `pauserecord.Excluded` and each visit's time is floored at zero, so no negative or double-counted time reaches the frame.
- **Test economy.**
  `TestMultiLLMProducer_MapsTheLeadResult` replaces the parity test rather than adding to it, and the deleted `SingleLLMProducer` tests' cancellation and error-exit behaviors are covered by the existing `TestMultiLLMProducer_Cancellation` and `TestMultiLLMProducer_ErrorExits`.
  The pattern-directive, refusal and twill cases of the deleted `Test*Spec` functions are subtests or rows of `TestDiscussionTable`, `TestDiscussionTable_Refuses` and `TestPlanTable`.
  I found no redundant new test and no behavior the diff adds without coverage.
- **Run.**
  `lyx gate test` over `loomengine`, `seatengine`, `shedadapters`, `shedrecipe`, `shedengine`, `loomrecipe`, `landingshed`, `darnengine`, `loomshed` and `shedbuild` passed.

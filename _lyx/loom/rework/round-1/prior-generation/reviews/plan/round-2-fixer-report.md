# Plan-Review round 2 — fixer report

Every finding was checked against the code and fixed in the plan.
`lyx loom validate-plan` passes over the edited plan, with only the two informational `create-new-unit` notes for card 4's new stencil files.

## Fixed

- **F1** (BLOCKING, card 7 leaves `TestWire_EverySeamFilled` red).
  Confirmed: `envkit.NilSeams` reports every nil func field, and `internal/loomcli/wiring_seams_test.go`'s `intentionallyNil` does not list `DiscussionTable`.
  The `DiscussionTable` fill in `internal/loomcli#loomCLI.wire` and the creation of `TestWire_DiscussionTableEvaluatesToExpectedShape` moved from card 8 into card 7, with the `internal/loomcli` budget raise.
  Card 7 can call `loomengine.DiscussionTable` and `seatengine.Prompts`, which cards 5 and 1 create.
  Card 8 now wires only `DiscussionSeats` and adds that assertion to the card-7 test as a subtest.
  The overview's `coverage-guard-bridge`, `no-parent-name-parameter` (applies to cards 5 and 7) and `test-budget-trajectory` decisions, the summary paragraph and the card index lines for cards 7 and 8 follow.
- **F2** (BLOCKING, `TestRecipeEngines_ReportsExactlyLoomsOwnEngineSet` not targeted).
  Confirmed against `internal/loomrecipe/recipe_test.go`: it compares `RecipeEngines()` element by element with `wantProducerTable`'s engine column.
  Card 8 now lists the test under `Edit:`, and its want set becomes that column plus `discussionSeatsEngine`, sorted.
  `RecipeEngines` stays sorted and de-duplicated.
  The overview's `coverage-guard-bridge` decision names the change.
- **F3** (MEDIUM, no card carries the gate-and-brief criterion).
  Card 5 lists `internal/loomengine#TestParentReviewBrief_NamesPathsAndSubmitLines` under `Edit:`, with an assertion that the brief names no `advisor-` path while `advisor-1.md` sits beside the two files.
  Card 7 lists `internal/loomshed#TestNewDiscussionGate` under `Edit:`, with a row that plants a failing `advisor-1.md` beside a valid record and support log and asserts the gate still passes.
  Card 7 already touches `internal/loomshed`.
  Both changes are rows or assertions in existing tests, so no budget changes.
- **F4** (LOW, the included stencil's single-agent framing and stale header).
  Card 4's advisors section now opens with a sentence that replaces the included single-agent framing, and a `wordingClaims` claim pins it.
  Card 4 also edits `contracts/stencils/loom/loom-template-discussion.md`'s leading header comment to name the chair stencil's include, filled through `stencil.FillWith`, as a second filler.
  `FillWith` strips the leading comment from both the main template and each include (`internal/stencil/stencil.go:36` and `:111`), so no prompt changes.
  The overview's `chair-stencil-includes-the-discussion-stencil` decision narrows "bytes unchanged" to "agent-facing body unchanged", and card 4's Intent and ImpactSummary follow.
- **F5** (LOW, card 8's false-branch subtest repeats existing coverage).
  The subtest now asserts only that the seat fake saw no table, and cites `TestNew_ShapeMatchesRecipe` and the sequence tests as the coverage for the rest.

## Disputed

None.

## Deferred

None.

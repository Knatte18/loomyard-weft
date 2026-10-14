# Plan round 2 — fixer report

## Changes

- **F1** (fixed as proposed): the premise holds against `internal/shedrecipe/recipe.go`, `contracts/specs/shed-recipe-spec.md` and `internal/loomrecipe/fixture_test.go`.
  Card 17's Intent gains a bullet rewording the `AnchorPath`, `WorktreeRoot` and `StencilsDir` doc comments to drop `SingleLLM` and name only the readers that remain.
  Its spec bullet now also swaps the `Config` bullet's `SingleLLMProducer` example for `MultiLLMProducer`.
  No targets change.
  Card 14, the last card to edit `buildSequenceFixture`, now rewrites that function's doc comment to describe the Discussion-Write, Plan-Write and Describe rows as running on the seat producer over `env.Seats`.
  `env.PlanSpec` joins the review's list of text to replace, since the Plan-Write paragraph names it too.
- **F2** (fixed as proposed, first alternative): card 5 now says card 4's unreadable-recipe frame skips the goto lines card 5 adds to the history-order list, so it still shows only D1's visit lines.
- **F3** (fixed as proposed, first alternative): card 8 now says `DiscussionTable` forms its placeholder `advisor-<n>` names with `seatengine.AdvisorName`, its last production reader until card 19.
  Only `RoleChair` and `RoleAdvisor` lose every reader in card 8.
  Card 20's ordering rationale stays as written.

## Disputed

None.

## Deferred

None.

---
round: 2
focus:
  - "Card order around the shedrecipe registry: `internal/shedrecipe/coverage_guard_test.go` `TestCoverageGuard` requires every registered engine to be reached by `loomrecipe.RecipeEngines()` (a fixed list of embedded sources in `internal/loomrecipe/names.go`) or `battenrecipe.RecipeEngines()`, or be allowlisted. Settle whether every card that adds a registry row also leaves `TestCoverageGuard` passing at its own card gate, and whether the recipe file and `RecipeEngines` reach sit in the same or an earlier card than the registration."
  - "Test economy in the sampler stencil card (`_lym/plan/03-sampler-stencil.md`): settle whether any planned assertion tests the stencil's fixed wording rather than a value `SessionSpec` computes, and whether an existing stencil-wide scan such as `TestStencils_NoUnboundedAskOperator` already covers the behavior."
  - "Doc and comment coverage of changed behavior: for each card that changes how a seam, recipe name set or timeout behaves (`landingshed.Deps` field docs for `CodeReviewOff` and `RerouteReviewed`, `loomrecipe.RowNames` doc, `internal/loomengine/template.yaml` `step_idle_timeout_min` comment, `internal/landingshed/handback.go` file comment), settle whether the text it makes false has a listed target on that card."
  - "Internal agreement of each card: settle whether each card's `ImpactSummary` names the same functions its `Intent` says are edited."
---

Round 2 re-reviews the plan after the round 1 fixes.

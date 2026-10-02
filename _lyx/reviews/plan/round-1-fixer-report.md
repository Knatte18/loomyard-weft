# Fixer report — plan round 1

All four findings are fixed.
`lyx loom validate-plan` over the edited plan returns `ok: true` with informational `create-new-unit` findings only.

## Fixed

- **F1 (BLOCKING): a status commit splits a rework round.**
  Added card 16 (`16-status-commit-holds-archive.md`) and its Card Index line.
  It gives `loomshed` a working-tree predicate, `ReworkArchiveInFlight`, in `generation.go`, and narrows `statusCommitPathspec`.
  While a rejection is pending and the highest round has no classed `record.json`, the status commit stages the status file and drive reports only, so the round commit is the one commit recording the archive.
  The card states its bound and carries unit and real-git tests.
  It is its own card because it can be tested on its own and card 5 is already large.
  The overview gains the Shared Decision `one-commit-per-round`, and `committed-round-carries-a-class` is reworded: the classed-only count remains the rule for a classless record that reaches HEAD outside the hold window.
  Card 5's restatement of that rule and its commit-pathspec sentence now point at card 16.
- **F2 (MEDIUM): `first_card` is checked outside `planparser`.**
  Rewrote card 7.
  `planparser` gains `CheckFirstCard` (check `rework-first-card`) in a new `firstcard.go`, outside both format entry points.
  `planglyph` gains `ValidateRework`, which is `ValidateFormat` plus that check converted blocking.
  `ValidateReworkPlan` computes the told number and calls it, so it no longer reads `plan.FirstCard` itself.
  `loom-plan-spec.md` documents the ID after the validation-checks list, which leaves the twenty-eight-row list and its counts unchanged.
  Tests were added for both new functions.
- **F3 (LOW): the generation-round rule has three wordings.**
  The overview's `generation-round-rule` now includes the class condition.
  It names card 9's HEAD reading as authoritative and says why the agents' working-tree reading agrees with it.
  Card 5 names the field (`class`, `exempt`/`required`).
  Cards 9, 12 and 13 state the rule with the class condition, and the rubric-test assertions in cards 12 and 13 now name `class`.
- **F4 (NIT): card 10's bound statement is thin.**
  Card 10's recipe-comment bound now states that the seam's inputs are agent-written, that the live cards are re-classified, and that a seam error falls back to a real review.

## Deferred

None.

# Fixer report — plan, round 1

## Changes

- F1 (MEDIUM), fixed as proposed: card 2's row "moved by a goto and then stepped is refused" now appends a `Plan-Write` goto entry, a `Plan-Write` stuck entry (`shedengine.Stuck`) and a second `Plan-Write` goto entry, with `CurrentProducer` at `Plan-Write` and `State` paused.
  A following sentence says the row isolates the every-entry-is-a-goto conjunct.
- F2 (LOW), fixed as proposed: card 2's `TestConsumeTacklerAttempt` paragraph now gains only the `wantHistory int` column; the arbiter test keeps its `spawn` column.
- F3 (LOW), fixed as proposed: card 4's Intent now has `hubPeersDeps` panic on a parse error of the embedded loom recipe, citing `TestRowNames_FollowEachRecipesRowOrder` (`internal/loomrecipe/recipe_test.go:245`), in place of the `logger.Warn` fallback.

## Disputed

None.

## Deferred

None.

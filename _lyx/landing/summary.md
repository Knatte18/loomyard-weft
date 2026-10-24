# Plan gate: project PATTERN headroom card by card

The plan gate now checks each pending card's own growth of `PATTERN.md` against the headroom line, as the card gate's `lint-pattern` step does.
Before, it validated only the final projected size, so a plan whose intermediate card grew the overview passed plan review and failed only at that card's gate.
A plan-review fix that moved a shortening card ahead of growth cards hit exactly this, and undoing it meant renumbering about twenty cards.

Closes #866

## What changes

- `internal/loomshed/patternplan.go` adds the blocking check `pattern-card-headroom`.
  It gives one finding per card not recorded done whose own step grows the projected overview past the headroom line, attributed to that card.
- The projection starts at the worktree's `PATTERN.md` and walks the pending cards in plan order.
  Each step applies the card's fenced entry lines with `pattern.ProjectOverview`, and `pattern.Headroom` measures the result against the overview before the step.
- The whole-plan `pattern-headroom` check is unchanged.
  It still catches growth already in the worktree against the merge base, which no pending card causes.
- Both headroom findings now share one way-forward text, `patternHeadroomWayForward`.
- `planparser.CardNamesPatternEntry` reports whether a card names an entry ID in backticks.
- `contracts/specs/loom-plan-spec.md`, `internal/loomshed/doc.go` and the plan stencil `loom-template-plan.md` describe the new check.
  The stencil tells plan writers that a later shortening card never covers an earlier card's growth, and that a card removing an entry names its ID in backticks.

## Design decisions

- A `removes` entry of `## PATTERN changes` is applied in the step of the first pending card whose text names its backticked ID.
  A removal no pending card names is applied before the first card.
  The plan ties a removal to no card, so this credits the removal in the way least likely to flag a plan its card gate would pass: the check can miss a failure, never invent one.
- A done card is never projected and never flagged; the worktree already holds its work.
- Cards run in plan card order, so each pending card's gate sees the overview the earlier cards leave.

## Not changed

- Renumbering helpers that rewrite prose card references; the renumber verbs report prose mentions as before.
- `pattern.Headroom`, `pattern.ProjectOverview` and `lyx loom lint-pattern`.

## Limits to know

- A card that edits `PATTERN.md` in prose, outside fenced entry lines, is invisible to both headroom checks.
- Crediting an unnamed removal before the first card makes a card that precedes the real removing card look smaller than its gate will measure it.

## Tests

`internal/loomshed/gates_test.go` rows cover:

- growth on an earlier card that a later card shrinks back;
- a card whose lines net to no growth;
- a removal credited through the card's prose;
- a removal no card names;
- a done card.

`internal/planparser/patternentry_test.go` covers `CardNamesPatternEntry`.
The suite was not run by hand here.

# Plan gate projects the PATTERN headroom card by card

The plan gate's `pattern-headroom` check measured only the overview the whole plan projects, against the merge base.
Each card's own gate runs `lyx loom lint-pattern`, which measures the card's working tree against the commit before it.
A plan whose earlier card grows `PATTERN.md` past the headroom line, with a later card shrinking it back, passed plan review and then failed at the fork's card gate, and fixing it there costs a renumber of the cards.

The new blocking check `pattern-card-headroom` closes that gap.
It starts at the worktree's `PATTERN.md` and walks the cards not recorded done, in plan order.
Each card's step applies its fenced entry lines with `pattern.ProjectOverview`, and `pattern.Headroom` measures the result against the overview before the step, the bound the card gate applies.
A card whose step grows past the line gets one finding naming the card and the same way forward as `pattern-headroom`.
The whole-plan `pattern-headroom` check stays as it was: it still catches growth already in the worktree against the base, which no pending card causes.

## Changes

- `internal/loomshed/patternplan.go`: `cardHeadroomFindings` and the `pattern-card-headroom` check, appended after `pattern-headroom`; the two headroom findings share one way-forward string.
- `internal/planparser/patternentry.go`: `CardNamesPatternEntry` reports whether a card's text names an entry's backticked ID, so the plan text stays read inside its sole parser.
- `internal/loomshed/gates_test.go`: rows for an earlier growing card that a later card offsets, a removal credited to the growing card, to a later card and to no card, and a done card that is never projected; rows that already grew past the line now also expect the card finding.
- Docs: `internal/loomshed/doc.go`, `contracts/specs/loom-plan-spec.md` and the plan writer's stencil `contracts/stencils/loom/loom-template-plan.md` describe the check and tell the writer to put a shortening on or before the growing card, and to name a removed entry's ID in backticks on the removing card.

## Design decisions

- A separate check name rather than a card attached to `pattern-headroom`, so the whole-plan cut against the base and a card's cut against its predecessor stay distinguishable.
- A plan's `removes` entry belongs to no card, so it is credited to the first pending card whose text names its backticked ID, and to the start of the walk when no pending card names it.
  Crediting an unnamed removal at the start can only miss a card gate failure, never invent one.
- A card that edits `PATTERN.md` in prose, outside fenced entry lines, stays invisible to both headroom checks, as before.
- The board entry also suggested renumbering helpers that rewrite prose card references.
  That is left out: a prose number may already mean the new numbering, so a rewrite can corrupt a correct reference.
  It is filed as board note E3, `renumber-rewrites-card-refs`.

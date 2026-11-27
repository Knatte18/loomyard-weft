# Hide the arras recipe from orchs and skills; motif is the design-first route

`hide-arras-recipe` · Note · enhancement, stencil, orch, board

## Seen (operator, 2026-11-27)

Orchs still propose `arras`, though it is never used.
The operator does not trust raddle's split enough to let a run split an entry on its own approval.

## Operator decision (2026-11-27)

Arras is no longer an option anyone is offered.
Its code stays, but nothing visible names it: the shipped orch role, the `ly:board` and `ly:board-policy` skills, help text and recipe listings.
Motif is the route for a design-first entry: it stores the decision record as a Sketch, and the Sketch's raddle run splits it on the operator's approval.

## Idea

- Rewrite the board-policy passage that proposes an arras task for a tapestry-sized note to propose motif.
- Drop arras from the recipe lists in the board skill, the orch role and `lym board` help.
- Decide whether a board write naming `recipe: arras` is refused or only warned.

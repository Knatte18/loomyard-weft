# A mechanical plan conflict is fixed in Webster by a small fixer agent, not by a plan reroute

`webster-tree-moved-fixer` · Task · enhancement, webster, loom, stencil · running · Drift-Verify

## Seen (beam-start-and-ownership, 2026-10-26)

`begin-batch 07` refused with `plan_conflict: tree_moved`, since card 16 still named two tests batch 06 had renamed.
Loom rerouted the whole run to Plan-Burler-Review: a new plan writer round and its perch, to fix two names in one card.

## Design

- A mechanical plan conflict such as `tree_moved` or `glyph-not-found` never reroutes to Plan-Burler-Review.
- A small dedicated fixer agent under the Webster run edits the named cards to match the tree, runs `lyx webster rebaseline --card NN` naming each card it edited, and Webster goes on.
- Only a conflict that changes what a card does goes back to the plan writer.

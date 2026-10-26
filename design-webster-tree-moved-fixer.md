# A renamed symbol a later card targets: catch it at plan and batch end, and fix it in Webster, not by a plan reroute

`webster-tree-moved-fixer` · Task · enhancement, webster, loom, stencil

Depends on: [`webster-stop-asked-once`](design-webster-stop-asked-once.md)

## Seen (beam-start-and-ownership, 2026-10-26)

Card 6 (`recipe-named-pawl`) declared the function renames `NewBeam` -> `NewPawl` and `BeamRouting` -> `PawlRouting` as Rename pairs, but listed the tests `TestNewBeam` and `TestBeamRouting` under Edit only.
Batch 06's implementer renamed the tests too, and its report said "No departures".
Webster warned after batch 06 that card 16 had rename candidates, but only as information.
`begin-batch 07` then refused with `plan_conflict: tree_moved`, since card 16 still named the old tests, and loom rerouted the whole run to Plan-Burler-Review: a new plan writer round and its perch, to fix two names in one card.

## Idea

- The plan writer handles it from the start: a card that renames a symbol also declares the rename of the tests named after it, and plan validation flags a Rename pair whose symbol has same-named tests left under Edit.
- The batch-end check is blocking: when a batch removes or renames a symbol a later card targets, the batch gets a departure while its context is fresh, and the implementer either keeps the name or declares the rename.
- A mechanical plan conflict such as `tree_moved` or `glyph-not-found` never reroutes to Plan-Burler-Review: a small dedicated fixer agent under the Webster run edits the named cards to match the tree, runs `lyx webster rebaseline --card NN`, and Webster goes on.
  Only a conflict that changes what a card does goes back to the plan writer.

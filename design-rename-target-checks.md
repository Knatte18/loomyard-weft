# Catch a renamed symbol a later card targets: in the plan writer, plan validation and a blocking batch-end check

`rename-target-checks` · Task · enhancement, webster, loom, stencil · running · Darn-Write

Depends on: [`webster-stop-asked-once`](design-webster-stop-asked-once.md)

## Seen (beam-start-and-ownership, 2026-10-26)

Card 6 (`recipe-named-pawl`) declared the function renames `NewBeam` -> `NewPawl` and `BeamRouting` -> `PawlRouting` as Rename pairs, but listed the tests `TestNewBeam` and `TestBeamRouting` under Edit only.
Batch 06's implementer renamed the tests too, and its report said "No departures".
Webster warned after batch 06 that card 16 had rename candidates, but only as information.
`begin-batch 07` then refused with `plan_conflict: tree_moved`, since card 16 still named the old tests.

## Design

- The plan writer handles it from the start: its stencil says a card that renames a symbol also declares the rename of the tests named after it.
- Plan validation flags a Rename pair whose symbol has same-named tests left under Edit.
- The batch-end check is blocking: when a batch removes or renames a symbol a later card targets, the batch gets a departure while its context is fresh, and the implementer either keeps the name or declares the rename.

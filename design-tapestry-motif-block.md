# Tapestry runs Discussion through a motif block and skips it for a discussed entry; motif runs alone; arras goes

`tapestry-motif-block` · Task · enhancement, shed, loom · running · Code-Bouncer

## Seen (2026-11-03)

The landing `e3ae7e25` added block rows, the recipe `motif` (tapestry's Discussion rows) and the recipe `arras`, which runs `motif` through a block row named `Cartoon` and then tapestry's rows from Drift-Baseline on.
Tests pin `arras`'s expanded rows equal to tapestry's, so `arras` is tapestry with nothing to bench against it.
`motif` is a block source only: it is not in `shedrun.RecipeNames`, so no task can run it alone.
The block row's name `Cartoon` takes the name the operator kept free for a finished plan and a later Plan block.

## Decision (operator, 2026-11-03)

Tapestry itself reads the board: when the entry is flagged discussed, it skips Discussion and starts at Plan from the stored record. No recipe beside it is needed.

## Expected

- Tapestry runs its Discussion through a `motif` block row in place of its own copy of the Discussion rows, so Discussion is defined once.
- The block row is named `Motif`, never `Cartoon`.
- The block's gate check passes over the block when the board entry is flagged discussed, reading the stored record into `_lym/discussion` so Plan-Write and the reviews read it as today; an entry not flagged discussed runs Discussion as today.
  This includes the passed-approval test reading a row inside a block, recorded in the decision record of `decision-record-on-board`.
- `motif` is a recipe a board task may name: a `motif` run runs Discussion-Write and its review and nothing else, then stores the record on the board and flags the entry discussed, and lands no code.
- `arras` is removed, with every place that names it by name (Seed-Child, Run-Shed, run reports, timing spans, recipe keys, `profile_recipe`).
- Tests pin tapestry's expanded rows to its rows before this change, config included, so the move into a block changes no run.

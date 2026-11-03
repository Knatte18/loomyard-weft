# Preflight refuses a seat stencil marker no value fills

A Darn-Write seat could start with a stencil marker, such as `subagent_model`, that no value filled.
The failure then surfaced as a hard failure inside the seat, as `seat "": fill stencil "darn-template-write": stencil: unfilled top-level marker(s): subagent_model`.
This change catches that case in Preflight, before any agent row spends tokens, and names the seat in every seatengine error.

Closes #17

## What changes

- `seatengine.Table.CheckMarkers(stencilsDir)` checks every seat's stencil, and every block it includes, for top-level markers no value fills.
  A marker is filled by a non-blank table or seat value, a name in `Optional`, or a marker the engine fills itself.
  The refusal names the seat, the stencil and every unfilled marker.
  It gives two causes with a way forward for each: the stencils are newer than the running `lym` binary, so re-step with a binary that fills them, or a value is blank, so give it text.
- `preflightshed` gains a `Checks` struct (`Batcher`, `Stencils`) and `NewPreflightWithChecks`, which replaces `NewPreflightWithBatcher`.
  A failing stencils check makes the Preflight row Stuck with the reason `seat stencils check failed: …`; a table that cannot be built reads the same way, not as an unfillable stencil.
- `shedrecipe.Env.PreflightStencils` carries the check to the Preflight row.
  `loomcli`'s `wireDarn` sets it for a recipe with a Darn-Write row, building the writer table the first spawn reads.
- Every seatengine error now names the seat by its strand role (`Table.seatLabel`), so the lead, which has an empty name, no longer reads as `seat ""`.

## Design decisions

- The `subagent_model` marker stays in the stencil and stays config-supplied; nothing hardcodes a model.
  The check guards the supply of the value rather than changing where it comes from.
- `Table.Validate` does not run the marker check.
  A resumed seat keeps the prompt it started with and never fills its stencil again, so a resume must not fail on stencils that changed since.
- Preflight checks only the Darn-Write writer table.
  The other seat tables are built from values known only at call time, which Preflight does not have.
- `composePrompt` is removed; `composeSeatPrompt` now takes the `Table`, so `Optional` and the seat label come from one place.

## To check by hand

- The root cause of the original run is not reproduced.
  The run ahead of the report landed on the day the marker was introduced, so a binary older than the stencils is the likely cause.
  To confirm, run `lym loom` against a hub whose stencils are newer than the binary and expect Preflight to stop with the version-skew refusal.
- Whether `lym loom` on a normal hub still passes Preflight with the new check was covered by unit tests only, not by a full run.

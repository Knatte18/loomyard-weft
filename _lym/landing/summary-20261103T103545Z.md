# Preflight refuses a darn stencil marker no value fills; name seat roles

A darn run halted at Darn-Write as a producer hard failure with `seatengine: seat "": fill stencil "darn-template-write": stencil: unfilled top-level marker(s): subagent_model`.
The value is not missing from the code: `wireDarn` passes `loom.yaml`'s `subagent_model` into `darnengine.DarnTable`, and `LoadConfig` refuses an empty one.
The stencils are read at call time from the hub's board copy, which a production binary refreshes forward.
The failing run started on a binary from before `subagent_model` existed, and a newer binary refreshed the board's `darn-template-write` mid-run, so the old binary filled a stencil holding a marker it had no value for.
This change makes that skew stop the run at Preflight with a way forward, and makes the error name the seat.

## What changes

- `seatengine`'s errors name a seat by its strand role, `SeatRole(prefix, name)`, so the lead, which carries no name, reads as the role prefix, `seat "darn"`, in place of `seat ""`.
  Name-validation errors still quote the name under test.
- `seatengine.Table.CheckMarkers(stencilsDir)` reads every seat's stencil and the blocks it includes and refuses the first seat with a top-level marker that no table value, seat value, `Optional` name or reserved marker fills.
  The refusal names the seat's role, the stencil, every unfilled marker and the stencils directory, and its way forward is a re-step with a lym binary that fills them.
  Stencil and block reading moves into one `readStencilSet` helper that `Validate`, prompt composition and `CheckMarkers` share; the test-only `composePrompt` wrapper goes.
- `preflightshed.NewPreflightWithBatcher` becomes `NewPreflightWithChecks(name, cwd, Checks{Batcher, Stencils})`.
  The stencils check runs after the batcher check and before `preflight.Check`; its failure is Stuck with `seat stencils cannot be filled: ` and the check's error, and is logged.
- `shedrecipe.Env` gains `PreflightStencils`, which the Preflight entry passes on, and `envkit` lists it as nil-legal.
- `loomcli.wireDarn` fills `PreflightStencils` for a recipe with a Darn-Write row: it builds the writer's table as a first spawn reads it, project directive included, and runs `CheckMarkers` over it.

## Design decisions

- `CheckMarkers` stays out of `Table.Validate`, which `Probe` and `Resume` also run: a resumed seat keeps the prompt it started with, so a stencil changed since then must not refuse its resume.
- The check is wired for the darn writer only.
  Its table needs only the run's told paths and no row output, so Preflight can build it before any row runs; the tapestry Plan and Discussion tables read row outputs and are left for later.
- A skew that happens after Preflight still fails at Darn-Write, now with the seat named; pinning a run to the binary and stencils it started with is the board note `run-pins-its-version`.
- The way forward names a binary that fills the markers rather than a command, since lym has no update verb and the binary comes from outside lym.

## To check by hand

- On a hub whose board `darn-template-write.md` gains a marker the running binary does not fill, start a darn run and confirm Preflight halts Stuck with `seat stencils cannot be filled: seatengine: seat "darn": stencil "darn-template-write" has marker(s) …` and the re-step way forward.

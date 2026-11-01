# Add Parts-Write and Parts-Plan-Write, a two-step plan beside Plan-Write

The plan writer is the costliest session of a tapestry run: it reads the code for the whole task in one session, and its context grows with the task.
This change adds two opt-in producers that write the plan in two steps, so a later research task can bench them against `Plan-Write`.
Nothing is benched here, and `Plan-Write` stays unchanged and stays the default.

## What changes

- `Parts-Write` reads the decision record and splits the task coarsely into parts, each with a contract and the shared spots no single part owns.
  It writes `_lym/parts/parts.md`, checked by the new `internal/partsparser` and a `parts` gate.
  Its reading depth is `light` or `medium`; heavy reading is not offered.
- `Parts-Plan-Write` builds a seat table from the parts file: one writer seat per part, each a fresh session that writes cards for its own part under `_lym/parts/<part>/`, and a chair that joins them.
  The chair numbers the cards into one sequence, sets cross-part dependencies, writes the shared spots as cards of their own and settles contracts with the writers.
  The joined plan passes the ordinary `plan` gate, so Plan-Review and Webster see an ordinary plan.
- Tapestry gains the two rows between `Drift-Baseline` and `Plan-Write`.
  They carry `entry_gate: tapestry.plan_split`, and `Plan-Write` carries `tapestry.plan_split != true`.
  With the flag off both new rows are recorded Skipped and the run is as before.
- New config keys in `internal/loomengine/tapestry.yaml`: `plan_split`, `parts_depth`, `parts_max`, and the model keys `parts`, `part_writer` and `parts_chair`.
  `parts_chair_timeout_min` defaults to twice the run's `plan_timeout_min`, and a set value not above it is refused at config load.
- New CLI: `lym loom validate-parts`, `lym loom validate-plan --part`, and the chair-only `lym loom join-parts`.
- The settled-verdict refusal of a goto onto a settled Plan-Review row now names the plan writer the run used.
- The plan format and glyph text moved into included blocks shared by `Plan-Write`, the part writers and the chair; `Plan-Write`'s composed prompt keeps its content.

## Design decisions to know

- `internal/seatengine` gains a writer seat kind: a seat that writes a product the chair joins, with the seat count taken from data at start.
  A table may not mix writers and advisors, and a writer table's lead must include `seat-directive-writer-chair`.
  A writer's failed end ends the step as Stuck, a resume tells the chair about a writer stopped mid-fix, and advisor tables behave as before.
- The join is a new write path in `internal/planparser`, the plan format's sole writer; `PATTERN-sole-parsers` is updated for it.
- Neither new row has `on_stuck`.
  `lym loom goto` moves a halted run only back, so the way back to a solo plan is to turn `plan_split` off and go to `Drift-Baseline`.
- Each row reads the flag when the run enters it, so the flag is meant to be set before a run starts.
  Flipped mid-branch, the rest of the run follows the new value.
- There is no review of the parts file; Plan-Review catches what a bad split causes.
- Neither producer forks lens subagents; `plan_lenses` stays `Plan-Write`'s.

## Check by hand

- Run a tapestry with `plan_split: true` end to end; the new stencils, the part writer and chair prompts, and the real tmux and creel flow between chair and writers were not run live.
- Seam quality: a part writer working from contracts may still need a neighbouring part's code; look for contract questions to the chair or seam findings in Plan-Review.
- The chair judges a card's position inside a shared spot's range; a misplacement is caught only where the card's refs show it (`uses-later-target`).
- The Plan-Review finding `card7-spec-extends-should-be-edit` (consistency, medium): card 7's spec says extends where it should say edit; confirm the fixer's resolution in the plan and the code it produced.
- The Code-Review finding `seatengine-lost-writer-end-step-untested` (scope, medium): the branch in `internal/seatengine` that ends the step when a writer is lost at resume; the fix pins it with subtests that fail by assertion, so check they cover it.
- The full-plan verify first failed on `cmd/lym.TestCLIForm_EveryCommand` because `validate-plan`'s `Long` exceeded the CLI form cap; commit `377fd58d` shortens it, so confirm that test passes.

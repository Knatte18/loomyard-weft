# Measure the batching effect on the Plan and Discussion writers

Adds `docs/research/batching-effect-audit-2026-11-01/`: a report, the scripts that rebuild it, and its tables.
It measures whether the batching sentences of `bf00a043` (2026-10-28) changed how the Plan and Discussion writers spend turns, context and money.
The change adds no code outside that folder, no agent run and no config.

## Finding

The transcripts show no batching effect, and the sample cannot show a small one.
After the change the Plan writer issues no more calls per turn (1.14 against 1.29) and its single-call turn share is higher (83% against 75%); its serial read and search turns are 44% of its turns against 41%.
The Discussion writer is level on every measure (1.17 calls per turn against 1.16; 69% single-call against 70%).
Context and cost are higher after for the Plan writer, but those runs are also larger tasks, and the report does not assign the gap to the sentences.
The 33-of-101 serial turns the turn audit found in the Plan writer are still there, so the plan-writer ideas of notes `plan-writer-context` and `loom-beam-darns` keep their value.

## What the folder holds

- `report.md` has the method, a per-session table for each writer, the group means, and a section on how far the groups compare.
- `run-all.sh` rebuilds every table.
  It runs a copy of the turn audit's `extract-turns.sh` beside its own `slugs.txt` and reuses that audit's `session-metrics.sh` for the serial-turn counts.
- `list-price.sh` prices each session at the `claude-opus-5-5` list price from `internal/shuttleengine/claudeengine/pricing.go` and assigns the group.
- `task-size.sh`, `writers-table.sh` and `group-means.sh` add task size and join the tables.
- `README.md` says how to rerun it and how to add a run.

## Design decisions

- A session is in the "after" group when the stencil's batching sentence is in its transcript, not by date.
  A date split would misplace a session whose run started before the commit but whose stencil was read after it.
  The sentence in shuttle's appended system prompt is not in a transcript, so the group does not confirm it reached the session.
- The `interactive-recipe` run has not landed, so it is shown in the tables but left out of every group mean.
- The price is fixed to `claude-opus-5-5`, the one model every writer session used, and `list-price.sh` stops on any other model instead of pricing it wrong.
- The sample is eight runs, seven of them landed: five before and two after.
  The report says so beside every mean and compares the one size-matched pair run by run instead of drawing a conclusion from the group means.

## Test coverage

The change adds shell scripts and data under `docs/research/` and no Go code, so `lym gate test --wait` ran only the tier1 guard, which passed.
The tables were rebuilt by running `run-all.sh` end to end on this machine.

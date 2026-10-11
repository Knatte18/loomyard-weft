# Burler: reviewer row, then fixer row

`burler-chair-late-spawn` · Note · enhancement, burler

## Seen (2026-10-11)

In the four newest loom runs (`.scratch/token-usage-by-role.md` from tools/tokencount), the Burler chair has the longest turn time of any role (22–30 min per run, 35–39% of all turn time) while costing less than the advisor.
Operator, 2026-10-11: the chair is spawned first and waits for the reviewer; that wait is likely cache waste, since the chair's prompt cache can expire before the review arrives.

## Idea (operator, 2026-10-11)

Run the reviewer first, then the fixer, so the fixer starts only when the review is ready and holds no turn open while it waits.

## Shape (operator, 2026-10-11)

The Burler round becomes two Shed rows instead of one seat table, sequenced by the FSM:

1. Burler-Review: a `MultiLLMProducer` with zero advisors whose chair is the reviewer and writes the review file.
2. Burler-Fix: a `MultiLLMProducer` with zero advisors whose chair is the fixer, reads the review file and fixes.

Each review segment's Bouncer `OnStuck` points at Burler-Review, Burler-Review goes on to Burler-Fix, and Burler-Fix returns to the Bouncer (`contracts/recipes/loom-recipe.yaml`, `internal/shedadapters`).
This keeps `MultiLLMProducer` as the one LLM producer, in line with `seats-replaces-singlellm`.

Rejected: keeping one seat table and having `internal/seatengine` defer the chair's spawn until the advisor delivers; it adds special code to the seat engine.

## Open

- The fixer can no longer ask the reviewer follow-up questions; check in the transcripts whether the chair uses that today.

An analysis agent is measuring the wait per round, whether the chair's cache expired in it, and the cost of the re-write (`.scratch/burler-cost-breakdown.md`, 2026-10-11).

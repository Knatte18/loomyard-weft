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

## Measured (analysis agent, 2026-10-11, `.scratch/burler-cost-breakdown.md`)

Over the four newest runs, Burler plus its Bouncer passes cost $46.8, about $11.7 per run: reviewer $24.5, fixer $18.1, Bouncer $4.2.

- The waiting itself is cheap: 57–61% of the fixer's turn time is `lyx shuttle await`, but its cache never expired in a wait (every await return was followed by a cache read with a 300–700 token write), so the wait cost $0.14 over 21 rounds.
  Reordering alone saves about $0.035 per run.
- The real waste is duplicate orientation: both seats load the same prompt, explore file (PATTERN overview, rubric, target, fasit), focus file, decision record and plan cards; about 44k of the fixer's ~64k tokens at its wait is that, $8.36 of its $18.10.
- In 31 archived rounds over six runs the fixer applied every finding and disputed or deferred none.

Lever with the largest saving: start the fixer as a fork of the finished reviewer, so it inherits the reviewer's context and cache, and run no fixer on an empty review; about $1.9 per run (16%).
The risk is the loss of the fixer's independent check, which changed no outcome in 31 rounds.
This changes the shape above: the fixer would be a fork of the reviewer, not a cold zero-advisor producer.
Operator, 2026-10-11: a fixer forked from the reviewer is what Burler had before, and it started with a large context it did not need (the reviewer's whole exploration).
So the fixer stays a fresh session, with two narrower levers:
- run no fixer on an empty review (Webster-Burler's opus fixer costs more than its Sonnet reviewer even when the review has no findings);
- give the fixer a leaner orientation than the reviewer: the review file and the targets it names, not the reviewer's explore file (rubric, fasit, PATTERN overview).
Operator, 2026-10-11: the fixer is meant to get a constrained fix report, exact instructions on what to change and where, since the reviewer has done most of the work; the measured fixer still loads the reviewer's full orientation, which is the waste to cut.
Operator, 2026-10-11: for Go code the tests carry Webster-Burler's weight, while a review is mostly useful for prose changes such as stencils; Operator decided later the same day: Webster-Burler runs on code anyway, as Sonnet reviewer plus Sonnet fixer (hub `loom.yaml` `webster_fix: sonnet[medium]`, set 2026-10-11), one round with auto-accept (`segment-budget-accept`); no prose-only gating.
Operator, 2026-10-11: no Sonnet reviewer; Millhouse experience says Sonnet is too weak for this.
Other levers measured: a Sonnet reviewer for Plan and Discussion (about $1.8 per run, highest quality risk, since every Discussion and Plan BLOCKING finding came from opus) and dropping Webster-Burler (about $1.4 per run); capping rounds is not recommended, since later Discussion rounds catch real BLOCKING findings.

An analysis agent is measuring the wait per round, whether the chair's cache expired in it, and the cost of the re-write (`.scratch/burler-cost-breakdown.md`, 2026-10-11).

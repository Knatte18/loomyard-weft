# Plan writer bench: twill against a solo Opus and a solo Fable on one plan

`plan-writer-context` · Task · research, loom, stencil · running · Preflight

## Idea (operator, 2026-11-12)

The Plan writer is the costliest session type (`docs/research/loom/writer-cost-remeasure-2026-11-08/report.md`: $17.05 a run for the writer and its forks).
The operator sees strong signs that the twill writer makes a far better one-shot plan in shorter time.
Tapestry is still used now and then, so the question is answered once, on one plan, and then closed.

## What it does

A bench, reported under `docs/research/loom/`, on one reference and three arms:

1. Reference: one tapestry task that has landed with a twill-plan Plan-Write, whose Discussion record and base commit can still be recovered.
   The task picks the most recent such task and names it in the report.
2. Arm A, twill: no new run.
   Its Plan cost, Plan rounds and Plan wall time come from the landed run (`lym report timing`, `go run ./tools/tokencount`), and its plan is the landed plan.
3. Arm B, solo Opus: the Plan writer on `opus[high]`, no twill directive and no forks.
4. Arm C, solo Fable: the Plan writer on `fable[medium]` (Fable 5.2), no twill directive and no forks.

Arms B and C each run once from the reference's frozen Discussion record, on the reference's base commit, by the setup of `docs/research/twill/plan-bench/twill-plan-bench-sonnet-medium-fable/report.md` "Setup".
Each is a hand-seeded pair stopped after Plan-Write, so it lands no code.

## Compare

- Plan cost, Plan rounds and Plan wall time per arm.
- Plan quality: the three plans graded against each other by the method of the earlier plan benches (repeated, partly, avoided), with keys drawn from the landed run's own Plan review findings.
  State the keys before reading the solo plans.
- One line of verdict: which arm makes the best plan, which the cheapest, and whether twill earns its cost.

## Cap

One reference, two new runs (one `opus[high]`, one `fable[medium]`), one report; cost cap $15; no repeat runs.
A run that fails to start is reported with its cause and not rerun.

## Not in scope

Changing any default and the `plan_split` producers.
The excerpt cut for the exploration fan is dropped: it is worth at most $5.95 of $46.94 (re-measure of 2026-11-08).

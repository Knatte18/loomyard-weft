# Batcher v2: Merriam context at fork time, a fitted model, plans in card order

`batcher-v2` · Task · enhancement, webster

Card batching (landed 2026-10-07) cut Webster's time and tokens visibly; this task makes its model measured instead of guessed.

## Merriam's context at fork time

Operator, 2026-10-07: the cost batchifier's `startup_context` (60000 in the `cautious` profile) is a fitted constant standing in for something that is not constant.

An implementer fork is a fork of Merriam, so it starts with Merriam's whole context at the moment it is spawned, and that context grows with every finished batch.
A constant is too high for early batches and too low for late batches of a long plan, which is where a large batch can overrun the budget.

The start context of batch N is Merriam's base context plus a growth term for batches 1 to N−1.

- Base: computable exactly before the run, since it is fixed by what Merriam loads: Claude Code's system prompt and tools, CLAUDE.md, the Webster stencil with the inlined PATTERN overview, and the plan files Merriam reads.
- Growth: what comes back between forks, the batch reports and the output of Merriam's `lyx webster` verbs; estimated, for example as a per-card or per-report allowance.

The partition stays computed once per run, as `webster-spec.md` requires.

Measure first: read Merriam's actual context at each fork from its transcript in the runs that used the `cautious` profile (`shuttle-turns`, `test-tiers`, `run-recovery`), to size the base and the growth term and to see how far the constant was off.
Compare weight per card in those runs against earlier identity-batchifier runs with `tools/tokencount`.

## Learning loop

Operator, 2026-10-07: the batchifier's model is a first guess; treat tuning as a learning loop rather than a design-time decision.

- Log: each batch records its profile, coefficients and per-card estimate components in webster's `state.json`; the fork transcripts give the actual peak context (max input+cache tokens of any message) and the actual weight.
- Analyse: extend `tools/tokencount -calibrate` to read those records from finished runs and fit the peak-context coefficients (estimate vs actual peak) and the cost side (weight per batch vs batch size).
- Decide on data: the batch size and budget that minimise weight per card without a rise in failed batches, recoveries or Webster-Review findings; quality outcomes (batch failed, recover-batch, review findings per card) belong in the same table.
- Re-fit as runs accumulate, and compare profiles across runs before changing the default.

## One flag for the weft repository

`tools/tokencount` has both `-weft` (finished runs) and `-history` (calibration) naming the same weft repository; one flag with one default would do.

## Plans run in card order

Rule (operator, 2026-10-06): a valid plan is executable strictly in card-number order; a card never depends on anything a later card does, and Webster follows that order as written and never sorts cards.

Plan-Gate refuses backward dependencies since `bugfix-webster` (`uses-later-target`, `delete-before-reference`), and card-batching made the batchifier's contiguous segments in card order the execution order.
Still open: `contracts/specs/loom-plan-spec.md` calls the card list a DAG of intent "not itself an execution order", and the plan template does not tell the writer the rule.
Both state it, to the plan writer and the plan reviewer, so Plan-Write stops bouncing on plans Webster used to reorder.

# Batchifier: a fork starts with Merriam context at fork time, not a constant

`batchifier-master-context` · Note · enhancement, webster

Operator, 2026-10-07: the cost batchifier's `startup_context` (60000 in the `cautious` profile) is a fitted constant standing in for something that is not constant.

An implementer fork is a fork of Merriam, so it starts with Merriam's whole context at the moment it is spawned, and that context grows with every finished batch.
A constant is too high for early batches and too low for late batches of a long plan, which is where a large batch can overrun the budget.

## Proposal

The start context of batch N is Merriam's base context plus a growth term for batches 1 to N−1.

- Base: computable exactly before the run, since it is fixed by what Merriam loads: Claude Code's system prompt and tools, CLAUDE.md, the Webster stencil with the inlined PATTERN overview, and the plan files Merriam reads.
- Growth: what comes back between forks, the batch reports and the output of Merriam's `lyx webster` verbs; estimated, for example as a per-card or per-report allowance.

The partition stays computed once per run, as `webster-spec.md` requires.

## Measure first

Read Merriam's actual context at each fork from its transcript in today's runs (`shuttle-turns`, `test-tiers`, `run-recovery`), to size the base and the growth term and to see how far the constant was off.
The fitting itself belongs to `batchifier-learning-loop`.

# Measure the darn writer's cost by token kind, activity and gate waits

Adds `docs/research/loom/darn-writer-anatomy-2026-11-08/`: a report, the scripts that produce its tables, and the tables.
No product code changes.
The measurement reads the session transcripts of 294 landed darn runs (writer, lookup subagents and cold gate fixer) and runs nothing.

## Findings

- Research, the writer's turns before its first edit, is the largest writer cost: $387.63 of $914.28 (42%), $1.32 a darn, mostly cache writes.
- The suspected cause, a gate run outlasting the five-minute cache, does not hold for plain gate runs.
  5 of 483 writer `lym gate` calls were followed by a cache rewrite ($7.81), and none of the 84 capped calls, the gate fixer's included.
- It does hold for the writer's wait on the cold gate fixer: 29 of 33 findings rounds after a fixer rewrote the whole context, $30.12 of the rounds' $42.31.
- All rewrites are 6.0% of all cost; the cache lifetime is five minutes in every transcript.
- Thinking is an estimated 58% of the writer's output.
  No run used another effort on Opus or Sonnet, so effort does not show in cost.
- Levers in order of saving, with the basis for each, are in the report: a Sonnet writer, thinking effort, ending the writer at a finished change, keeping the cache warm over the fixer wait, and research.

## Design decisions

- The 600-second window that ties a fixer to the writer prompt after it, and the 20,000-token floor for calling a turn a rewrite, are in `gate-rounds.sh` and `turn-costs.sh`.
- The savings rows are estimates priced at the sample's mean fixer round; the report marks the rows that are ceilings.
- The large per-turn tables are committed so the report's tables rerun without the transcripts.

## Testing

Nothing to test: the change adds a report and read-only scripts.
`summary.sh` was rerun after fixing two joins that double-counted rows, and its output is what the report quotes.

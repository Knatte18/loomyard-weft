# Measure how Plan and Discussion writers explore and use lookup subagents

Adds `docs/research/writer-lookups-measure-2026-11-04/`: a report, the tables behind it and the scripts that rebuild them.
It reads the transcripts of landed tapestry runs and measures, per Plan-Write and Discussion-Write session, the exploration turns, their chains, their context and cost, and the lookup subagents started.
No agent run, replay or bench was made, and no production code changes.

## Result

- The sample is 7 writer sessions of 3 tapestry runs, not the 10 runs asked for: only 4 tapestry runs landed after 2026-11-02, and one ran no writer.
- The 3 Plan writers followed the stencil guidance, each handing one batch of 3 or 4 `sonnet` subagents to the Agent tool in one message.
  1 of 3 full Discussion sessions did.
- Exploration is 46% of the writers' cost ($11.03 of $23.76), and 15% of it is paid above 200k context.
  Every chain run above 200k is in a Plan session, after its one lookup batch.
- Moving every search turn to subagents at the observed subagent price would save about $4.7, one fifth of writer spend, as an upper bound.
  For the short late chains, batching in the writer saves more than a subagent does.
- The report recommends no bench of the guidance and no stronger stencil now, and a re-run once more tapestry runs have landed.

## Design decisions

- Reused `docs/research/turn-audit-2026-10-28/extract-turns.sh` unchanged through a copy beside this run's slugs, as `turn-savings-measure-2026-11-04` does, and added `writers.sh` and `subagents.sh` for what that extraction lacks: per-turn cost, chains, the context line and the subagent join.
- Priced turns at the `claude-opus-5-5` list price from `claudeengine/pricing.go`; the Plan sessions sum to the Plan writer figure `tokencount` gives for the same runs.
- Kept the sample at what landed instead of widening it to runs before 2026-11-02, since the guidance the entry asks about was in force only for these runs.
- Left the `turn-savings-measure-2026-11-04` directory untouched, as `batching-hook-reach` edits a script in it.

## To check by hand

- Run `docs/research/writer-lookups-measure-2026-11-04/run-all.sh` and compare the tables with the committed ones; it needs the transcripts of the three runs on this machine.
- The classification of a Bash call as a lookup, and the dependency mark, are the turn audit's heuristics, applied unchanged.

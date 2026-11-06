# What makes runs slow and what would make them faster: deep analysis

`repo-size-run-time` · Task · research, report, loom · running · Open-IDE

## Question

What makes runs slow, and what can be done to make them faster?
This is an open analysis: find where a run's time goes and why, follow the evidence wherever it leads, and rank what to change.
No single theory is the premise, and the candidates below are starting points to test, add to and throw out.

The run's depth is the point: its writer is Fable at high effort, by the operator's word.

## Known so far

`docs/research/speed/task-runtime-before-after-2026-11-06` is the starting point, with its `runs.tsv`, `producers.tsv` and scripts.
It found darn code runs slower since 2026-11-01 (active time 31.7 to 45.0 min on average, the fitted fixed cost per run up from 27 to 41 min), tapestry level per 100 changed lines, and no saving per run from project `turn-savings` although turns per card fell.
It could not separate the causes.
Check its figures before building on them.

`docs/research/turns/` holds the per-turn audits, and `docs/research/tokens/` and the rest of `docs/research/speed/` what earlier work measured; link them rather than repeat them.

## Scope and cap

- Read recorded data only: no agent run, no new trial, no change to lym, and no test suite beyond what a timing of a single command needs.
- Inputs: `lym report timing`, `cost`, `convergence`, `lym gate report`, the archive tags and run records, the systemd journal, and the session transcripts of landed runs, with the tools under `tools/`.
  `lym report` runs from the hub's prime only, so run it with `env -C` on the prime; it writes nothing.
- The run-level figures cover every landed run in the records.
- Transcript-level work reads at most 40 landed runs, darn and tapestry both, picked so size and recipe are matched across the periods compared; the report states the pick and why.
- Read lym's code wherever a cause needs explaining: a row's real work, what a stage waits on, what is serial and could run side by side.

## What to find

Account for a run's wall time as a whole, so the parts add up to the run and the part no one can explain is shown as such.
For each recipe, and for a typical darn and tapestry run, split the time into what the agents think (model time per turn), what their tools do, gates, waits for a slot or lock, the hand-over between rows, and session starts.

Candidates to test, none of them assumed:

- The repository's size, which is the operator's suspicion: Go source on main grew about 23% from 2026-10-27 to 2026-11-06.
  Does a search or edit cost more, do agents read more per changed line, is the start load larger, does the gate grow with the tests?
  A first look shows the darn writer's daily median not following that growth, and review and gate time rising; test this properly.
- Recipe rows: what each row costs, and whether its result pays for it, with Record-Write, Drift rows, Describe and review in mind.
- Parallel load: runs in flight rose from 1.8 to 3.4 on average; effect on the machine, gate slots, API speed and rate limits.
- Model speed: time per turn and per output token by role, model and effort, and whether a role uses a stronger model than its work needs.
- Prompt cache: misses after waits, and the time and tokens they cost.
- Context size: time per turn against context, and late turns at large context.
- Serial work that could run side by side, and fixed latency at each row: driver turns, polling intervals, dobby pickup, locks, pushes, fabric work.
- Review rounds and fixer loops: what a round costs, how often it finds something that matters.
- The machine: the two machines the hub has run on, and any file-scanning cost per new binary or opened file.
- Anything else the data shows.

## Output

`docs/research/speed/run-slowness-causes-2026-11-06/report.md`, with its scripts and tables beside it, and a line in `docs/research/README.md`.
The report opens with the answer:

- The causes, ranked by minutes per run, for darn and for tapestry, each with the evidence and how sure it is.
- What to do: a ranked list of changes, each with its estimated saving in minutes per run, its cost to build, and what it risks in quality.
  Separate what could be done with config alone from what needs code, and name quick wins.
- What the data cannot settle, and the cheapest measurement that would settle it.

State plainly when a saving is an estimate.
The changes are proposals in the report; the operator decides what gets built.

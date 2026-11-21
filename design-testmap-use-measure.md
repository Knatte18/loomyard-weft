# Measure whether the reach and mutation reports change what runs land

`testmap-use-measure` · Task · research, testmap, gate · starting · Open-IDE

## When

Release on or after 2026-11-21 (operator, 2026-11-19), once at least 10 darn runs have landed since `mutation-report-step` (6dbc7a05).
The board has no start date; the orch releases it by hand.

## Question

Since 6dbc7a05 every darn gate writes `reach.json` and `mutation.json`, a reach turn can send the writer back for unreached functions,
failure reports carry triage lines, and the code review rubric gets `reach_report` and `mutation_report` markers.
Do they change what lands?

## Method, exact cap

The 10 most recent darn runs landed after 6dbc7a05, one pass each, no repeats, no tests run.
For each run, from its records (gate reports, reviews, fixer reports, decision record, commits):

- reach turns taken, and whether the writer added tests after one;
- mutation survivors reported, and how many a review raised as a finding and a fixer acted on;
- triage lines in failure reports, and whether a fixer used one to call a failure unrelated;
- time and spend the reach and mutation steps added to the gate, where the records show it.

## Output

A report under `docs/research/speed/testmap-use-measure-<date>/report.md` with a table per run and a verdict:
keep the steps as they are, tune them, or turn some off.

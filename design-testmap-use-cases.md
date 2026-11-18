# Find real uses for the per-test coverage maps

`testmap-use-cases` · Task · research, testkit, gate · done

## Why

lym builds per-test coverage maps (`internal/testcover`, built by `lym gate test --map`): for each tier 0 and tier 1 test, the code blocks it runs.
Today they have almost no job:

- Test selection from the maps was retired on 2026-11-15 (`testmap-on-command`, `72b26d1a`): it saved the gate 0 to 4.2 s against 7.7 to 31 min of upkeep per landing.
- `lym gate redundant` confirms redundant tests by mutation, but 27 of 30 sampled confirmations were false
  (`docs/research/speed/redundancy-confirmed-survey-2026-11-16/report.md`).
- The gate report's advisory redundancy step on a diff's new tests is the only use left.

The operator is sure the maps are good for something. Find what.

## Scope, exact

- One writer session, model Fable 5.2 (set in this entry's `run_settings`), no other model.
- Search the web: research and industry practice on per-test coverage (test impact analysis, regression test selection, fault localisation such as spectrum-based methods,
  test prioritisation, flaky-test triage, mutation testing guidance, change-risk scoring, code ownership, review aids, documentation of behaviour), and tools that use it.
- Read lym's own map format and the reports named above, so each idea is judged against what the maps hold today and what they cost to build
  (`docs/research/speed/coverage-map-measurement-2026-11-18/report.md`: a full build took 12 min 50 s over 210 packages).
- Think in particular about lym's agents: a writer, reviewer or fixer that changes a function, meets a failing test or reviews a diff.
- Runs: at most one `lym gate test --map` build of one package, to check a concrete idea against real data. No other test runs.

## Output

A report under `docs/research/speed/testmap-use-cases-<date>/report.md`:

- every use found, each with its source (a URL or paper for outside ideas), what it would need from lym, a rough build cost, and the expected gain;
- a ranking, with the top three argued in full, including why each beats doing without the maps;
- a plain statement for any use that is not worth it.

No code changes.

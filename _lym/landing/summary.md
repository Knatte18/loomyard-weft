# Report every package's redundancy candidates after one timed map build

Adds a bench report under `docs/research/speed/redundancy-candidates-survey-2026-11-16/` and its raw output under `data/`.
No production code, no test and nothing under Bolt changes.

## What it does

One `lym gate testmap` over every package ran in this worktree at its own HEAD, with the maps kept under its `.lym/`.
It took 7 min 2 s of wall and 25.1 CPU-minutes, built 212 packages and failed none.
`lym gate redundant` then ran once per mapped package.

118 packages hold 1,026 candidates, 6.43 s at tier 0 and 5.32 s at tier 1 of standalone time.
The report reads the five largest packages and finds each sampled candidate asserting something its covering tests do not.
It estimates the cut at about 1 percent off a full run of tiers 0 and 1, and recommends against filing it as a speed task.

## Design decisions

- Per-test build timings and the failed-test reasons are what the build's envelope reports; the report states what it does not carry.
- The seconds the payoff uses are standalone seconds, which bound wall time from above.
- `internal/fabricengine` is listed and not read.

## Test and verification

No test or code changed, so there is nothing for a gate to run beyond the existing suite.
The figures are reproducible from the files under `data/`.

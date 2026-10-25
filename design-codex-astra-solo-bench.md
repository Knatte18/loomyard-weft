# Bench Codex Astra against solo Opus as a plan reviewer, four runs

`codex-astra-solo-bench` · Task · feature, codex, burler · running · Finalize

## Goal (operator, 2026-10-25)

Compare Codex's strong model with Claude's strong model as a solo reviewer, like for like: `gpt-6-astra` against solo Opus, both with no lens forks, on the same plans with the same prompt.
Follows `docs/research/codex-review-bench-2026-10-25/report.md`, where Astra was cut off by the plan's 5-hour limit before the plans, and Claude's reference was Opus with a lens fan, not solo.

## Exact cap

Four runs in all, no more:

| Input | Astra, effort medium | Opus solo, effort medium |
|---|---|---|
| `P1` (`gate-runs-once` plan) | 1 run | 1 run |
| `P4` (`machine-handover` plan) | 1 run | 1 run |

No other models, efforts or repeats, and no smoke-test runs beyond what the first real run shows.
The inputs, snapshots, prompt and `codex exec` line are the earlier bench's (`data/harness/` in its report directory); Opus runs the same reviewer prompt solo, with the solo cluster rule and no forks.

## Before the first Codex run

The Codex account is ChatGPT Plus, for testing and benching: read the 5-hour window's `used_percent` first, and stop and report if a refused or cut-off run shows the limit.

## Output

A short report in `docs/research/codex-astra-solo-bench-<date>/report.md`: per input, the findings each found, shared, unique and real on reading, false, the verdicts, set against the recorded Claude fan review; tokens, wall time and the share of the 5-hour window.

# Bench Codex as a Burler reviewer against Claude's round-1 reviews

Adds `docs/research/codex-review-bench-2026-10-25/`: a report, and the data it rests on, on whether Codex can review loom discussions and plans as well as the Claude reviewers.
No production code, stencil, config or PATTERN entry changes; the diff is text under that directory only.

## What was measured

Ten finished loom runs (six discussions, four plans) from the history of `loomyard-weft`, each reviewed again by `codex exec` over the production burler reviewer prompt, solo, at the commit the Claude round saw.
Three models ran at medium effort: `gpt-6-luna` on all ten inputs, `gpt-6.1-sol` on all ten, `gpt-6-astra` on the first five.
Each Codex finding was graded against the Claude findings of the same round: shared, codex-only real, codex-only unchecked, or false.

`report.md` holds the summary table and the conclusion:

- On this sample Codex does not reach the Claude fan's coverage.
  The best setting, `gpt-6.1-sol` at medium, reproduced 16 percent of Claude's findings and a third of its `BLOCKING` ones, and added six findings the Claude round did not raise, one of them checked against the code.
- `gpt-6-luna` is fast and cheap but approved three of the four plans Claude blocked.
- `gpt-6-astra` matched Sol's coverage on its five inputs at about four times the plan cost.
- A Codex seat is defensible only as an extra, non-gating reviewer at `gpt-6.1-sol`.

The report also covers how `codex exec` behaves headless under the ChatGPT login and what each run cost against the plan's two rate-limit windows.
`data/` carries every review file Codex wrote, the per-run usage, the grading of every finding and the scripts that staged and ran the bench.

## Design decisions

- **The bench stopped on the plan's 5-hour limit.**
  The grid script had no check for the record's 70 percent stop, so the window passed 79 percent after Astra's fourth input and the limit refused the runs from the sixth.
  Astra's `D6` and four plan inputs, every effort above medium and every repeat run are unmeasured, and the report says so.
- **One run per cell.**
  No variance is measured, so close settings cannot be ranked.
- **The grading did not use each round's fixer report or bouncer verdict as evidence.**
  The record planned to read them; the report states the grading is recall against Claude's list, not against ground truth, and lists 12 unchecked Codex findings.
- **The code commit is the main branch at the round's start,** not the run's fork point.
- **No hash of `~/.codex/config.toml` was taken before the bench.**
  Its unchanged state rests on its file time, which predates the bench; every run used a scratch `CODEX_HOME`.

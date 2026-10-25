# Bench Codex as a Burler reviewer against Claude reviews of finished runs

`codex-review-bench` · Task · feature, codex, burler

## Goal (operator, 2026-10-25)

Bench Codex as a Burler reviewer before any of note `codex-engine` is built: Codex reviews the discussions and plans of finished loom runs, many times, and its findings are set against what the Claude reviewers found in the same runs.
Codex spends its own tokens; Claude only drives the bench.

## Method

- Inputs: discussions and plans of landed loom runs, with their Claude review reports, from the history of `loomyard-weft` (`_lyx/reviews/discussion`, `_lyx/reviews/plan`), each reviewed at the commit the Claude round saw.
- Codex runs headless (`codex exec`) with the review stencil as prompt, over several models and efforts (Luna, Sol, Astra).
- Per input and setting: findings Claude also found, real findings Claude missed, false findings, Claude findings Codex missed, tokens and wall time.
- Also tested: `codex exec` itself, and billing under a ChatGPT login (plan rate limits per run).

## Output

- A report in `docs/research/codex-review-bench-<date>/report.md`, with a summary table and a conclusion: whether Codex reaches the Claude reviewers' quality, and at which model and effort.
- No production code.
- Codex runs with its own `CODEX_HOME` under the worktree's `.scratch/`, linking `~/.codex/auth.json`; the operator's `~/.codex` is never changed.
- Stop early and say so in the report if the plan's rate limits come close to running out.

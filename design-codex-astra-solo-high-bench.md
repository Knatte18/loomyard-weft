# Bench solo Codex Astra at high effort on one plan

`codex-astra-solo-high-bench` · Task · research, codex, burler · running · Worktree-Create

## Question

How much does effort alone change a solo Codex Astra plan review?
`docs/research/codex-astra-solo-bench-2026-10-25/report.md` ran solo Astra at medium effort on `P4`: 4 findings, 2 of the Claude fan's 11, none of its 3 `BLOCKING` ones, in 166 s.

## Cap

- One input: `P4`, the `machine-handover` plan, with the snapshot, solo prompt and grading of the solo bench.
- One model and effort: `gpt-6-astra` at high effort.
- One `codex exec` call, no repeat, no Claude run.
- Stop and report what was done if a rate-limit window passes 70 percent.

## Method

Reuse the solo bench's harness under its `data/harness/` unchanged except for `model_reasoning_effort="high"`.
Record the wall time, fresh input, output and reasoning tokens, and the windows.

## Grading

Grade every finding against the fan reference of `P4` as the solo bench did: `shared`, `unique-real` or `false`, in `data/match.tsv`.
Report beside the solo bench's medium-effort Astra row and the Claude fan row.

## Output

`docs/research/codex-astra-solo-high-bench-<date>/report.md` with its `data/`, landed as the run's commit.

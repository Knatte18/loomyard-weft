# Bench a Codex Astra lens fan against the Claude fan on one plan

`codex-astra-fan-bench` · Task · research, codex, burler · running · Drift-Verify

## Question

Does a Codex Astra lens fan find what Claude's lens fan finds on a plan review, where solo Astra did not?
The solo bench, `docs/research/codex-astra-solo-bench-2026-10-25/report.md`, found that solo Astra reproduced 4 of the fan's 23 findings over `P1` and `P4` and none of its 5 `BLOCKING` ones.
This bench gives Astra the fan's shape: lens forks and a handler, as Claude's round 1 had.

## Cap

- One input: `P4`, the `machine-handover` plan, 17 cards, whose Claude fan reference holds 11 findings, 3 of them `BLOCKING`.
- One model and effort: `gpt-6-astra` at medium effort.
- One fan: 3 lens reviews and 1 handler review, one run each, so 4 `codex exec` calls in all and no repeat.
- No Claude run: the Claude fan reference is the one the earlier benches recorded.
- Stop the bench, and report what was done, if the 5-hour window passes 70 percent.

## Method

- Reuse the solo bench's harness under its `data/harness/` and its `P4` snapshot from `data/inputs.txt`, with a scratch `CODEX_HOME` linked to the operator's `auth.json`; the plan is now ChatGPT Pro.
- The 3 lenses are the lenses Claude's round 1 on `P4` used, each with its own prompt from that round's stencils, and each run as a separate `codex exec` over a fresh copy of the snapshot.
- The handler run gets the round-1 handler prompt with the 3 lens outputs in place of its forks' results, and writes the round's verdict and findings.
- Record per call the wall time, the fresh input tokens and the 5-hour and weekly windows, as the solo bench did.

## Grading

Grade the handler's findings, and any lens finding the handler dropped, against the fan reference of `P4` the way the solo bench graded: `shared`, `unique-real` or `false`, in `data/match.tsv`.
Report beside the solo Astra and Claude fan rows: findings, fan findings found, `BLOCKING` found, verdict, wall time, tokens and window points.

## Output

`docs/research/codex-astra-fan-bench-<date>/report.md` with its `data/`, landed as the run's commit.

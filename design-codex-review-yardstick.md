# Codex code-review bench: the decision record as yardstick

`codex-review-yardstick` · Task · enhancement, shuttle, codex · running · Drift-Baseline

## Question

Does one reading direction that names the decision record and the acceptance criteria as the yardstick let the forked Codex Astra fan find the Claude Code-Review fan's findings it rejected?
`docs/research/codex-code-review-bench-2026-10-28/report.md` found that both Codex runs of each input rejected the fan's record findings by name, on the reading that the board snapshot, not the decision record, is the measuring stick: `F1` (`BLOCKING`) and `F2` on `spawn-text-file-busy`, and the code-against-record findings `F3`, `F5` and `F6` on `fixer-overcap-review`.

## Change

Exactly one change to that bench's setup: one added line in `data/prompts/block-v2-code.md` stating that the decision record's decisions and the acceptance criteria are the yardstick the diff is judged against, beside the board entry, and that a divergence from either is a finding.
Read the rubric and target text the harness renders first to see what made the runs read the board snapshot as the only yardstick, and word the line against that; record the line and the reason in the report.
Everything else stays as that bench ran it: the harness under `docs/research/codex-code-review-bench-2026-10-28/data/harness/`, the same two inputs and reviewed commits, the same reference findings, context parity, the fork cap of 20, `gpt-6-astra` at `medium` for handler and forks.

## Cap

Exactly 2 `codex exec` handler runs: each of the 2 inputs once, at medium effort.
No other Codex run, no high effort, no reruns of a failed run beyond one retry of a run that failed to start.

## Grading

Grade as that bench did, against the same reference, and set each run beside that bench's two runs of the same input: fan findings found (and `BLOCKING`), real findings the fan lacks, false findings, verdict, fresh input tokens and wall time.
Name for each of the record findings above whether the run found it, and whether the added line drew any new false finding.

## Output

`docs/research/codex-review-yardstick-2026-10-28/report.md` with the changed block and raw outputs under `data/`, in the shape of that bench's report.

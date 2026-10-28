# Decision record: codex-review-yardstick

## Goal

Measure whether one added reading line, naming the decision record's decisions and the acceptance criteria as the yardstick beside the board entry, lets the forked Codex Astra fan find the record findings the earlier bench's runs rejected: `F1` and `F2` on `spawn-text-file-busy`, `F3`, `F5` and `F6` on `fixer-overcap-review`.

## Scope

- Two `codex exec` handler runs, one per input, `gpt-6-astra` at medium for handler and forks.
- The harness, inputs, reviewed commits, reference findings, context parity and fork cap of 20 are those of `docs/research/codex-code-review-bench-2026-10-28/`.
- The one change is one added line in a copy of `data/prompts/block-v2-code.md`.
- Output is `docs/research/codex-review-yardstick-2026-10-28/report.md` with the changed block, harness copies and raw outputs under `data/`.
- No code under `internal/` or `cmd/` changes.

## Decisions

### D1 — The yardstick line is worded from the rubric and target text the runs read

The line is worded after reading the rendered target, fasit and rubric text that made the earlier runs take the board snapshot as the only yardstick.
The report records the line and the reason.

### D2 — The earlier bench's harness is copied and re-pointed, not edited in place

The harness scripts are copied under the new bench's `data/harness/` with the root path changed to this worktree's scratch tree.
The earlier bench's files stay unchanged.
The only prompt difference is the added line; the build asserts that.

### D3 — Run budget is exactly two handler runs

One run per input, medium effort.
A run that failed to start is retried once; no other rerun, no high effort.

### D4 — Grading follows the earlier bench's rule

A Codex finding matches a reference finding by the earlier bench's matching rule.
Every unique finding is judged real or false by reading the code.
The report lists, for each of the five named record findings, whether the run found it, and whether the added line drew a new false finding.
Each run is set beside the earlier bench's two runs of the same input on found, `BLOCKING` found, unique real, false, verdict, fresh input tokens and wall time.

### D5 — The report states its limits

One run per input and one grader support no rate; the report says so.
Differences from the earlier runs inside the earlier bench's own run-to-run spread are not called an effect of the line.

## Constraints

- `PATTERN-lym-temp-root` and the scratch rule: staging, scratch `CODEX_HOME` directories and clones live under `.scratch/` and are removed by literal path afterwards.
- The scratch `CODEX_HOME` links the operator's `auth.json`, as the earlier bench did.
- `docs/` change only, so no test run is needed; `CLAUDE.md` exempts text no build or test reads.
- Markdown uses one sentence per line.

## Auto-mode assumptions

- The earlier bench's staging is reproducible: its weft commits `abfabfcf6` and `c9fb5eb56` and code commits `7615d129` and `d50e5277` are reachable in this hub.
- `codex` 0.164.0 is signed in on this machine.

## Open risks

- Sampling noise: two runs may differ from the earlier runs by chance alone.
- A start failure outside the single retry leaves a cell empty; the report then says so.
- A line that names the record may draw false findings against the writer's own claims.

## Acceptance criteria

- D1: the report quotes the added line and states the reason it was worded that way, citing the rendered text it answers.
- D2: the only diff between the new `block-v2-code.md` and the earlier one is the added line, and the earlier bench's directory is unchanged.
- D3: `data/reviews/` holds exactly one run per input and the report states two handler runs.
- D4: the report has the comparison table with the earlier bench's two runs per input, and a per-finding table for `F1`, `F2` on `spawn-text-file-busy` and `F3`, `F5`, `F6` on `fixer-overcap-review`, plus a statement on new false findings.
- D5: the report has a Limits section naming run count, grader and spread.

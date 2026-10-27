## Goal

Measure the tuned forked Codex Astra fan, with the Claude reviewers' context, on the plan `P1` (`gate-runs-once`) at high effort, and record it beside the earlier `P1` rows.
The deliverable is `docs/research/codex-astra-tuned-p1-high-bench-2026-10-27/report.md` with its `data/`, landed as the run's commit.

## Scope

- In: one `codex exec` handler on `P1` at `gpt-6-astra` high effort, spawning 3 lens forks with `fork_turns="all"`; one run, no repeat, no Claude run.
- In: the tuned-prompt bench's harness, context and `data/prompts/tuned-block.md` reused unchanged, with only the paths and the input changed.
- In: grading of the handler's findings, and any dropped fork finding, against the Claude fan's 12 `P1` findings, in `data/match.tsv`.
- Out: no production code, no PATTERN change, no change to any earlier bench's files.

## Decisions

### Harness copy

Copy the tuned-prompt bench's `data/harness/` and `data/prompts/tuned-block.md` into the new `data/`, changing only the `ROOT` and harness paths to this worktree and the new bench directory.
Rationale: the task requires the setup unchanged; the scripts hold no other path.

### Input and lenses

Build `P1` from `data/inputs.txt` of the solo bench (weft `c36025a89`, code `ab038b4e8`).
Use the lenses the fan's round 1 recorded in `round-1-usage.yaml`: `affected-set`, `guard-reach` and `way-forward`.
`P1` carries a round-1 focus file; `build.sh` already routes it into the explore instruction, so it stays in.
Rationale: same lenses and inputs as the reference, so the rows compare.

### Stop conditions

Check the rollouts for three forks with `fork_turns="all"`, overlapping spans, `forked_from_id` of the handler, and an `AGENTS.md` load in each; stop and report if any fails.
Read the weekly window before and after the call; stop and report if it passes 70 percent.
Rationale: both are the entry's caps.

### Grading

Grade each handler finding and each dropped fork finding as `shared`, `unique-real` or `false` against the 12 fan findings and the snapshot's cards and code.
State for each of the fan's `BLOCKING` findings `F1` and `F2` whether this run found it.
Rationale: the earlier benches' method.

### Sibling load

Siblings `codex-astra-tuned-p1-med-bench` and `codex-astra-tuned-p4-med-bench` may draw on the same account; the report says so beside the window figures.
No retry or repeat is made for window attribution.

## Constraints

- High effort for the handler and its forks; never `xhigh` or above.
- Scratch work under `.scratch/`; only the report and `data/` are committed.
- No raw module-wide test run; this change touches no Go code.
- Markdown with one sentence per line.

## Auto-mode assumptions

- The operator's `~/.codex/auth.json` is linked into a scratch `CODEX_HOME`, as in the earlier benches.
- The grader is this Claude session, as in the earlier benches.

## Open risks

- One input, one run and one grader: no variance is measured.
- Fork prompts are encrypted in the rollout, so that each fork held the block rests on the cluster rules.
- Recall is against the fan's list, not against the truth.

## Acceptance criteria

- `report.md` states the conclusion, a summary table with the earlier `P1` rows, per-call wall time and fresh input and output tokens, the windows with the sibling note, the fork evidence, the grading table and the limits.
- `data/` holds inputs, usage, rollout evidence, context hashes, `match.tsv`, reviews, prompts and harness.
- The rollout evidence shows three concurrent forks that inherited the handler's context and an `AGENTS.md` load in each rollout.
- Each of the fan's `BLOCKING` findings on `P1` is marked found or not found.

# Bench: the forked Codex Astra fan as Code reviewer on two darn diffs

`codex-code-review-bench` · Task · research, codex, burler · running · Finalize

## Question

Can the forked Codex Astra fan stand in for the Claude Code-Review fan on a darn diff?
Every Codex bench so far reviewed plans (`docs/research/codex-v2-p1-bench-2026-10-27/report.md` and its siblings); none reviewed code.
Darn runs are most of the work now, and the Code-Review reviewer with its forks is about 18% of a darn run's spend (`codex-engine` note, 2026-10-24 measure).

## Inputs

Two darn runs that landed on `main` on 2026-10-28, chosen by these rules:

- profile not `docs`, and a diff of real Go code;
- the run's Claude Code review returned at least one finding, so there is a reference to grade against;
- the two differ in the packages they touch.

The reference is the Claude Code-Review fan's findings of the round that reviewed the diff, read from the run's records.
The diff under review is the one that round reviewed, checked out at the commit it reviewed.

## Setup

- The forked Astra fan as the plan benches ran it: one `codex exec` handler that reads the target and spawns the lens forks through `spawn_agent` with `fork_turns="all"`, with the fork cap of 20 tool calls.
- Lenses: the Code-Review lenses the Claude fan ran on that round, from the shipped stencils, in place of the plan lenses.
- Reading directions: block v2 from `docs/research/codex-v2-p1-bench-2026-10-27/`, with any line that only fits a plan dropped and the reason noted.
- Context parity: the PATTERN overview inlined and the snapshot's `CLAUDE.md` loaded as `AGENTS.md`, as a Claude reviewer of the same round gets it.
- Model `gpt-6-astra`, effort `medium` for handler and forks.
- Reuse the harness under `docs/research/codex-v2-p1-bench-2026-10-27/data/harness/`, adapted; build nothing in `internal/`.

## Cap

Exactly 4 `codex exec` handler runs: each of the 2 inputs twice, all at medium effort.
No other Codex run, no high effort, no reruns of a failed run beyond one retry of a run that failed to start.

## Grading

One grader, the writer, matches each Codex finding against the reference findings as the plan benches did, and records per run: reference findings found (and `BLOCKING` ones), Codex findings that are real but not in the reference, false findings, the verdict against the fan's verdict, fresh input tokens and wall time.
The two runs per input give a first measure of run-to-run spread.

## Output

`docs/research/codex-code-review-bench-2026-10-28/report.md` with the harness and raw outputs under `data/`, in the shape of the plan bench reports.

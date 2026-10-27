# Bench a Codex Astra fan with real forks at high effort on P4

Adds `docs/research/codex-astra-fork-fan-bench-2026-10-27/`, a report and its data for one bench: a single `codex exec` handler on the `machine-handover` plan (`P4`) that spawns three lens forks through `spawn_agent` with `fork_turns="all"`, `gpt-6-astra` at high effort throughout.
No code changes; the commit adds only the report directory.

## Result

The forked fan found 1 of the Claude fan's 11 findings on `P4` and none of its 3 `BLOCKING` ones, and returned the fan's verdict (`BLOCKING`) with 9 findings: 1 shared, 8 real and absent from the fan's list, none false.
That is no recall gain over the cold-call Astra fan (0 of 11) or solo Astra (2 of 11).
It took 737 s and 299k fresh input tokens, against 907 s and 466k for the cold-call fan.
`report.md` holds the summary table beside the earlier four rows, the per-call figures and the deviations.

## Fork evidence

The rollouts show three forks started 8 s apart, running concurrently for about 2.5 minutes, each a child of the handler's thread and each beginning with the handler's reading in its first request (about 147k input tokens, 145k of them cache hits).
The run was therefore not a fallback to cold calls.
`data/rollout-evidence.txt` holds the figures.

## Design decisions

- The Agent-tool wording of the code commit's fan cluster rules became `spawn_agent` with `fork_turns="all"` in `data/harness/build-forkfan.sh`; the fork boilerplate keeps its inherited-context sentence and its cap of 6 tool calls.
- The stable `multi_agent` feature, on by default in Codex CLI 0.164.0, was enough: a probe of two short calls spawned forks with and without `features.multi_agent_v2`, and the run set no flag.
- The handler's own tokens and the forks' are separate in the rollouts, so the table adds them.
- No fork finding was dropped by the handler, so `data/match.tsv` holds only the handler's 9 findings.
- The rate-limit window moved from 3 to 8 percent, far from the stop at 70 percent.

## Limits

One input, one run and one grader measure no variance.
The effort differs from the earlier Astra rows (high against medium), so the table cannot separate the fork shape from the effort.
The fork prompts are encrypted in the rollouts.

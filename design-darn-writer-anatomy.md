# Measure what the darn writer spends on: writing, research, thinking, gate waits

`darn-writer-anatomy` · Task · research, darn, loom · done

## Why (operator, 2026-11-08)

`docs/research/loom/darn-cost-2026-11-08/report.md` found the darn writer and its lookup subagents are 58% of a darn ($3.13 of $5.45).
The operator asks what in the writer costs: writing, research, thinking, or gate waits.
A suspected cause: a gate run outlasts the 5-minute prompt cache, so the writer's next turn rewrites its whole context to the cache.
lym already guards against this: a capped `lym gate test` call answers `done: false` at the caller's keep-warm cap and the agent calls again, and a key whose runs are long gets the long cap (`internal/gatecli/doc.go`, Capped call). The measurement checks whether that guard holds in practice, including while the cold gate fixer runs.

## Scope

No agent run, no test and no gate run: this reads session transcripts, run records and code only, as `tools/tokencount` and the 2026-11-08 report did.
Inputs: the darn writer sessions of every darn run landed from 2026-10-25 to the start of this run.

## Measure

Per writer session, at API list price, and summed over the sample:

- Cost by token kind: cache writes, cache reads, fresh input, output, and of output the thinking share where the transcript shows it.
- Cost by activity: turns before the first edit (research), editing turns, turns spent on gate runs and their fixes, and the lookup subagents.
- Gate waits: each tool call that ran a gate or test, its duration, and whether the turn after it paid a cache write of the whole context (a gap past the cache lifetime); the cost of those re-writes in total and per darn.
- Gate failures: how many verify-gate rounds a darn needs, how often the existing cold gate fixer (`gate_fixer`, `internal/darnengine/template.yaml`) ran and what it cleared, and how many writer turns gate fixes still took.
- The writer's wait while the gate and the cold gate fixer run: its length, and whether the writer's turn after the fixer paid a cache write of its whole context; the cost of those re-writes, apart from the re-writes after a plain gate run.
- The cache lifetime the writer sessions actually get (5 minutes or longer), read from the transcripts' cache fields.

## Questions to answer

- Which part of the writer is the largest cost.
- Whether cache re-writes after gate waits are a large share, and what a writer that ends at a finished change, leaving every gate failure to a cold Sonnet fixer, would have saved on these runs (an estimate from the transcripts).
- The writer runs `opus[medium]` by default; how many sessions ran at another effort or model, and whether effort shows in the cost.

## Report

Write `docs/research/loom/darn-writer-anatomy-2026-11-08/report.md` with per-run tables, the sample size stated, and the levers in order of what they would save.
Commit the report and no code.

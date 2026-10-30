# Gate runs spend much of their time starting processes: start fewer

`gate-fewer-launches` · Note · enhancement, gate

## Seen (operator and hub orch, 2026-10-30)

Runs are slow on OSL-1046, gates above all, and every process start there is expensive.
`lym gate report --slug pr-review-empty-branch` shows `launch_seconds` of 266 of 639 s wall for `code-burler-fix` and 153 of 516 s for `darn-write`, with a wait for a gate slot of about zero: the time is in the test runs, and a large part of it in starting processes.

## Idea

The aim is a faster gate run, by starting fewer processes per run.
- Find what a gate run launches: how many test binaries, `go` invocations and helper processes per tier, and which of them repeat from one gate key to the next.
- Cut the launches: link several packages' tests into fewer binaries, or reuse a built test binary across the gate keys of one run, wherever the result stays the same.
- Measure a gate run before and after, so the gain is known; the measurement runs as a darn task with profile `docs`.

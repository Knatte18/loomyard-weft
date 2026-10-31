# Gate runs spend much of their time starting processes: start fewer

`gate-fewer-launches` · Task · enhancement, gate · draft

## Seen (operator and hub orch, 2026-10-30)

Runs are slow on OSL-1046, gates above all, and every process start there is expensive.
`lym gate report --slug pr-review-empty-branch` shows `launch_seconds` of 266 of 639 s wall for `code-burler-fix` and 153 of 516 s for `darn-write`, with a wait for a gate slot of about zero: the time is in the test runs, and a large part of it in starting processes.

## Cause (operator, 2026-10-31)

OSL-1046 runs Cortex XDR, which holds back a binary with a hash it has not seen, a newly built one, for about 10 s before it runs.
A process start of a known binary is cheap; the cost is each new binary a gate run builds and launches.

## Idea

The aim is a faster gate run on OSL-1046, by building and launching fewer new binaries per run.
- Find what a gate run builds and launches: how many test binaries per tier, which are rebuilt with a new hash from one gate key or run to the next, and how many `go` invocations and helper processes start.
- Cut the new binaries: link several packages' tests into fewer binaries, or reuse a built test binary across the gate keys of one run, wherever the result stays the same.

## Measurement (operator, 2026-10-31)

The measurement runs on Neve, not OSL-1046, as a darn task with profile `docs`: the count of new binaries and launches is the same on both machines, since the code and tiers are the same.
The gain on OSL-1046 is computed from that count at about 10 s per new binary.
Exact cap: two gate runs on Neve, one `--upto 2` and one of the whole suite, each measured once.

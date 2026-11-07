# Time every internal/fabricengine test one by one on a quiet machine

`fabricengine-per-test-times` · Task · research, fabric, testkit · running · Code-Bouncer

## Why (operator, 2026-11-07)

Eleven tasks of project `fabric-test-speed-work` cut `internal/fabricengine` from 57 s to about 42-44 s, short of the 36 s its Sketch expected (`docs/research/speed/fabric-test-speed-remeasure-2026-11-07.md`, `docs/research/speed/test-targets-bench-2026-11-07.md`).
No one has measured which of its tests take the time now.
The operator wants every test of the package timed one by one, on a quiet machine, to see which are slow.

## Quiet machine

- Before each pass, check `/proc/loadavg` and `lym loom peers`: no other run may be in flight and the one-minute load must be below 1.0.
  If not, wait and check again, up to 30 minutes in all; past that, run anyway and state the load in the report.
- Record the load and runs in flight before each pass.

## Runs (exact cap)

Run each once, in the foreground with the Bash tool timeout 3600000, and no other run:

1. `lym gate test --tier 0 ./internal/fabricengine -- -count=1 -run . -parallel=1`
2. `lym gate test --tier 1 ./internal/fabricengine -- -count=1 -run . -parallel=1`
3. `lym gate test --tier 2 ./internal/fabricengine -- -count=1 -run . -parallel=1`

`-parallel=1` makes each test's elapsed time its own work, not time spent waiting for other parallel tests.
`-run .` keeps the gate from carrying an earlier green record over the unchanged tree.
Read each test's elapsed time from the `go test -json` events the gate records (`lym gate report` or the run's JSON log).

## Report

Write `docs/research/speed/fabricengine-per-test-times-2026-11-07.md`:

- Every test and subtest of the package with its tier and elapsed time, sorted slowest first, and the package wall time of each pass.
- The share of the package time the slowest ten tests take.
- For each of the slowest fifteen, from reading its code: what it spends the time on, such as hub builds, git processes, binary starts or fixed waits.

Commit the report and no code.

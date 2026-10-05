# Test suite several times slower than its August benchmark

`test-suite-regression` · Note · bug

The test suite has slowed several-fold since its last recorded measurement.

| Machine | Untagged | Integration | Smoke | Date |
| --- | --- | --- | --- | --- |
| Ryzen, Linux bare metal (`docs/benchmarks/test-suite-timing.md`) | 3.60 s | 17.18 s | | 2026-08-13 |
| Ryzen, Kubuntu (operator, idle machine) | 13 s | 60 s | 1.6 s | 2026-10-05 |

Untagged is about 3.6× and integration about 3.5× slower on comparable hardware.
On the Intel 155U laptop under WSL2 (benchmark 9.0 s / 34.9 s), a full verify (build, both tiers, smoke vet) took 8–9 minutes per gate on 2026-10-05, partly from four parallel runs sharing four cores.
Every loom run pays it several times: each Burler round, the Webster verify gate and Publish run the full command.

- Run `go run ./cmd/testtiming -full` on an idle machine and record the result in `docs/benchmarks/test-suite-timing.md`, with its slowest-tests table.
- Find what landed since 2026-08-13 that the slowest packages and tests point to, and restore the floor; the benchmark doc lists the earlier levers (hermetic git env, real-time-wait seams, tier purity).
- Related: `test-tiers` (which tier each gate runs).

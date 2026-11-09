# Measure whether running fabricengine tier 1 as several shards cuts its wall

`gate-shard-slow-package` · Task · research, gate, fabric · running · PR-Review

## Seen (orch, 2026-11-08)

`internal/fabricengine` tier 1 is the wall of every tier 1 pass that includes it: 40.8 s of a 56.5 s whole pass (`docs/research/speed/fabric-test-cuts-remeasure-2026-11-08.md`).
Its tests run as one `go test` process, and the package runs only about three times faster than serial on a 12-core machine (`docs/research/speed/fabricengine-per-test-times-2026-11-07.md`).
The two test cuts dropped 348 tests and saved only about 4 s, because the dropped tests ran in parallel with others: the test count is not what holds the wall up.

## Idea

`lym gate test` runs a slow package as several processes at once, each taking a share of its top-level tests through `-run`, so the package's wall falls toward its serial time divided by the shard count.
It needs no change to the tests and serves any slow package.
Open for a later build task: how a package is chosen (a threshold on its recorded time, or a list), how many shards, and how the green record and the failure report merge the shards.

## This task: measure first (operator, 2026-11-08; load decision 2026-11-09)

A darn `docs` run on `internal/fabricengine` tier 1, run on the work machine OSL-1046, whose Cortex XDR scan of new binaries and files slows git-heavy tests.
The machine is never quiet: the operator works on three other projects there at the same time, all using Loomyard with their own suites, mostly C#.
That load is the machine's normal use, so the measurements run under it and are representative of it, and no quiet machine is waited for.
Each measurement records the load beside its result: the load average before and after the run, and the processes busy at the time.
Measure, once each:

1. The share of the package's tier 1 time spent in tests that are not marked parallel, and the longest serial chain, from one `-json` run's events.
2. The package's wall as one process, and at two and four shards, each shard a `-run` regex over a share of the top-level tests, the shards started together: three runs in all.
3. During the one-process run of item 2, whether the package is bound by CPU or by waiting: the CPU use across the run (for example `/usr/bin/time -v` or sampled `/proc/stat`), the disk wait, and the count of git processes and hub copies.
   Most tests already call `t.Parallel()` (55 of the package's test files have none, 9 of them set environment variables), yet the package runs only about three times faster than serial;
   say whether more parallel tests, shards, or fewer git processes and hub copies would move the wall.

The report lands under `docs/research/speed/` with the scripts beside it, states the machine and the load of each run, and says whether sharding is worth building.
No code or test changes.

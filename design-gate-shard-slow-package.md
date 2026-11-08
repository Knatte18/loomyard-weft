# lym gate test runs a slow package as several processes, each taking a share of its tests

`gate-shard-slow-package` · Note · enhancement, gate

## Seen (orch, 2026-11-08)

`internal/fabricengine` tier 1 takes 132 s serially and about 45 to 48 s in a gate pass on a 12-core machine, about three times faster than serial, and it is the wall of every tier 1 pass that includes it.
Its tests run as one `go test` process, where tests not marked parallel run one after another.

## Idea

`lym gate test` runs a slow package as several processes at once, each taking a share of its top-level tests through `-run`, so the package's wall falls toward its serial time divided by the shard count.
It needs no change to the tests and serves any slow package.
Open: how a package is chosen (a threshold on its recorded time, or a list); how many shards; how the green record and the failure report merge the shards.
Measure first: the share of the package's time spent in tests that are not parallel, and the wall at two and four shards on a quiet machine, once each.
The test count study (`fabric-test-redundancy`) has landed (`b121cc02`, report under `docs/research/speed/fabric-test-redundancy-2026-11-08/`): of 590 top-level tier 1 tests, 370 cover only blocks another test also covers, as reading candidates and not cuts.
The cuts themselves are the task `fabric-test-cuts`, still running, so measure after it lands: fewer tests may make this smaller.

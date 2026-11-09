# Measure whether sharding internal/fabricengine tier 1 cuts its wall

Adds a measurement report and its scripts under `docs/research/speed/fabricengine-shards-2026-11-09/`.
No code or test changes.

## What it finds

On the work machine under its normal load, running the package as two shards took 249 s against 242 s for one process, and four shards took 292 s.
The report does not recommend building sharding on this evidence.

- 59.5 percent of the one-process wall is a serial phase: 46 top-level tests not marked parallel, in a package of 400.
- The package was bound by neither CPU nor disk: its process tree used 1.23 cores of 16, the machine was 57 percent idle and iowait was 1.5 percent.
- The package started about 65 thousand git processes, and each took longer as more ran at once (6.8 ms, 8.0 ms, 10.6 ms at one, two and four shards).
- Hub copies are about 3 percent of the spawns.
- The report names fewer git processes in the serial phase and fewer serial top-level tests as the levers, and states what was not measured: a quiet machine.

## What it adds

- `report.md`: the answer, the method, each run's load and the busy processes, and the three items' tables.
- `shard-runner.sh`: the `-exec` program that runs the test binary as one process or N shards and samples the machine.
- `serial-share.sh`: the serial share and longest serial chain from a run's test2json events.
- `resources.sh`: CPU, disk and git-process figures from a run's samples and trace files.

## Design decisions

- The runs go through `lym gate test` with go test's `-exec`, because a raw `go test` is refused.
  Each shard run is one gate tier 1 run holding one gate slot.
- The shards' split uses the per-test times of `fabric-test-redundancy-2026-11-08/pertest.tsv`, which are serial times from a quiet machine, and the report names the imbalance that followed.
- The decision record's `## Goal` sets the measurement under the machine's normal load, so no run waited for a quiet machine.
  Each run was done once, and the runs' start loads differ.

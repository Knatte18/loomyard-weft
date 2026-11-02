# Measure fabricengine tier1 with the test work directory on /dev/shm against the default

`fabricengine-shm-bench` · Task · research, gate, fabric · draft

## Seen (orch, 2026-11-02)

`test-time-audit-3` in `docs/benchmarks/test-suite-timing.md` measured the full suite on OSL-1046, an Intel Core Ultra 7 366H with Cortex XDR.
`internal/fabricengine` took 348.72 s of the 388.6 s `tier1` wall, against 45.7 s on the Ryzen machine without endpoint security.
The operator's reading is that Cortex scans every file the `fabricengine` tests create on disk.

## Question

Does pointing the test work directory at `/dev/shm`, in memory, bring the `fabricengine` `tier1` time on this machine close to the Ryzen machine's?

## Method

- Run only the `internal/fabricengine` package in `tier1`, with the test binary cache bypassed (`-count=1`), through `lym gate test` as the gate rule in the system prompt says.
- Two settings: the default temp root, and the temp root on `/dev/shm`.
  The writer reads how lym and the tests choose their temp root (`internal/lymtemp`, `TMPDIR`) and uses the setting that moves every file the tests create; it says in the report which setting it used and what it checked to see that the files really landed in `/dev/shm`.
- Interleave the runs, default, shm, default, shm, default, shm, so a drift in machine load hits both settings alike.
- Read the package's own wall from the test output, and `/dev/shm`'s use before and after each shm run, since memory is the cost.
- Record the machine's load average at each run, and the revision and the Go version.

## Bounds

At most six runs: three with the default temp root and three with `/dev/shm`, no more and no cold run outside them.
One package, one tier, one machine.
No code change and no config change: the setting is made for the run only, and the report says whether a permanent setting is worth a task of its own, as a note.

## Report

A section in `docs/benchmarks/test-suite-timing.md` after `test-time-audit-3`, with the six walls, the memory use, the load averages, the median of each setting, and a plain statement of whether `/dev/shm` helps.
It lands as a commit, so the run's worktree is the only change.

## Timing

The run is only meaningful with the machine otherwise idle: the operator pauses the other runs with `lym tender pause` before it starts and resumes them after, and the writer makes no pause of its own.

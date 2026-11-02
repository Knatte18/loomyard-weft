# Measure fabricengine tier1 with the test work directory on /dev/shm against the default

`fabricengine-shm-bench` · Task · research, gate, fabric · done

## Seen (orch, 2026-11-02)

`test-time-audit-3` in `docs/benchmarks/test-suite-timing.md` measured the full suite on OSL-1046, an Intel Core Ultra 7 366H with Cortex XDR.
`internal/fabricengine` took 348.72 s of the 388.6 s `tier1` wall, against 45.7 s on the Ryzen machine without endpoint security.
The operator's reading is that Cortex scans every file the `fabricengine` tests create on disk.

## Question

Two things, and only these:

1. Can the test work directory be `/dev/shm`, in memory, for the `fabricengine` `tier1` run on this machine: do the tests pass there, and does memory hold?
2. Does it avoid Cortex's scan, shown by the package's wall under `/dev/shm` against the default under the same machine load?

How near it comes to the Ryzen machine's absolute time is no part of the question.

## Method

- Run only the `internal/fabricengine` package in `tier1`, with the test binary cache bypassed (`-count=1`), through `lym gate test` as the gate rule in the system prompt says.
- Two settings: the default temp root, and the temp root on `/dev/shm`.
  The writer reads how lym and the tests choose their temp root (`internal/lymtemp`, `TMPDIR`) and uses the setting that moves every file the tests create; it says in the report which setting it used and what it checked to see that the files really landed in `/dev/shm`.
- Run the settings in three pairs, the two runs of a pair started together, so both meet the same load from the rest of the machine, other runs of the hub included.
  Each pair needs two gate slots at once: when the gate holds two free slots at once the pair starts; otherwise the writer waits for them, and never runs the two of a pair one after the other.
- Alternate which setting starts first in each pair, since the faster run ends first and leaves the other a lighter machine.
- Read each run's package wall from the test output, and `/dev/shm`'s use before and after each shm run, since memory is the cost.
  The machine holds 30 GB and `/dev/shm` is 16 GB, shared with the other runs of the hub: when `/dev/shm`'s use passes 6 GB during a run, the writer stops the runs of the pair, reports the peak it saw, and goes on to the report with the runs it has.
- Record the machine's load average at each run, and the revision and the Go version.
- The two runs of a pair slow each other, so the absolute walls are no measure of a lone run: only the ratio of the two walls of a pair counts.

## Bounds

At most six runs: three with the default temp root and three with `/dev/shm`, no more and no cold run outside them.
One package, one tier, one machine.
No code change and no config change: the setting is made for the run only, and the report says whether a permanent setting is worth a task of its own, as a note.

## Report

A section in `docs/benchmarks/test-suite-timing.md` after `test-time-audit-3`, with the six walls by pair, the ratio of each pair, the memory use, the load averages, whether every run passed, and a plain statement of whether `/dev/shm` can be used and whether it avoids the Cortex slowdown.
It lands as a commit, so the run's worktree is the only change.

## Timing

No pause of the hub is needed: the pairs run beside the other runs, and the pairing cancels their load.
The writer makes no pause of its own.

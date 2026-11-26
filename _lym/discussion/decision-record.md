## Goal

```markdown
Imported from [issue #258](https://github.com/Knatte18/loomyard/issues/258).

## Seen

Gates and tiers run several times slower on one machine than on another, and it is not clear why.
Measured on a work laptop with Cortex XDR against a bare-metal Linux machine without endpoint security:

- The tiers of this repository took 118 s, 522 s and 112 s for tier0, tier1 and tier2 on the laptop, against 18.8 s, 58.8 s and 50.0 s in `docs/benchmarks/test-suite-timing.md`.
  The suites differ in age, so the two are indications only.
- A new test binary is held about 3 s on its first run in a folder with no exception, and 0.02 to 0.05 s in one with an exception.
- Tier1's wall time is bound by one package, `internal/fabricengine`, at about 400 s, whose tests start thousands of `git` processes.
  One test, `TestTakeOverRun`, starts about 2460 processes in 15 s.
- `git --version` took 5.2 ms on the laptop and 1.9 ms on the other machine, while `/bin/true` took 1.3 ms and 1.6 ms.
- Synced writes, file writes and fsync do not explain it: a profile of that test shows 22 000 `write` calls in 0.3 s, and a git commit loop is as fast with `core.fsync=none`.

The measurements so far were done by hand, with different commands on each machine, so the two machines cannot be compared on one footing.

## Idea

Add a small program under `tools/` that measures what a test run is made of, prints one comparable report, and runs the same way on every machine.

- Process starts, from Go with `os/exec`: `/bin/true`, `git --version` and the lym binary, with the median per start in microseconds, and the user and system time of the children.
- The first run of a binary with a new hash: build a tiny program with `-ldflags=-B=0x<random>` into each folder named by a `--dir` flag, and time its first and second run.
  Give it one folder with an exception and one without, so the hold and the effect of an exception show as two numbers.
- File operations in each `--dir`: create, write, close and delete a small file, `stat`, `mkdir` and `rmdir`, and a 4 KiB write with and without fsync.
- A git workload shaped like `internal/fabricengine`'s tests: init a bare repository, clone it, commit, and push, repeated, with wall, user and system time per step and the number of processes started.
- A machine descriptor in the report: the machine name, CPU model and core count, kernel, governor and mean frequency, the throttle count, the load average and CPU pressure before and after, the Go and git versions, and whether `/opt/traps` is present.
- One machine-readable report per run, and a compare mode that prints the ratio between two reports.
- It deletes every folder it makes, and it makes none outside the `--dir` folders and its own temp folder.

Notes:

- Build it as a binary an operator can run; `go run ./cmd/testtiming` is refused to an agent session, so a plain `go build -o` is the safer shape.
- Record the load, since neither machine is ever calm; a result without the load is not comparable.
- The result goes into a measurement report under `docs/benchmarks/`, for each machine, beside `gate-test-xdr.md`.
- Run it with `--dir` set to a folder under `/tmp/lym-*` and one under `/tmp/zz-*` on the laptop, and with the same two on the other machine, so one run tells whether a folder exception works.

## Earlier note on the same aim (`cortex-cost-measure`, 2026-11-09)

### Seen (operator, 2026-11-09)

Cortex XDR on OSL-1046 holds each new binary for about 10 s and scans each file a test creates, and fabricengine tests create many.
The shard report (`docs/research/speed/fabricengine-shards-2026-11-09/report.md`) put the 242 s one-process wall down to load, but the 2026-11-08 figure of 37 to 41 s was measured on another machine, so the gap is machine and Cortex, not load.
The package used 1.23 cores of 16 and `pmd` and `piud` were busy in nearly every sample.

### Idea

Separate the Cortex cost from the rest: the time a freshly built test binary waits before it runs, the time file creation takes with and without a scanned folder, and the fabricengine tier 1 wall with the repository and temp folders excluded from the scan, if the operator can have that changed.
The result says whether to cut tests, cut file and binary creation, or ask for an exclusion.
```

## Scope

## Decisions

### Added after Discussion (arbiter, 2026-11-26): The docs/benchmarks/ measurement reports follow the operator's runs, outside this change

Decision: This change ships the tools/machinecost program and its tests only. The per-machine measurement reports under docs/benchmarks/, beside gate-test-xdr.md, are written from the operator's runs of the built binary on the laptop with Cortex XDR and on the other machine, each with one -dir under /tmp/lym-* and one under /tmp/zz-*, after the program lands. No fixer adds a docs/benchmarks/ report in this run, and no reviewer raises the missing reports as a finding again (round-1 review F3, ledger key benchmark-reports-omitted-without-deferral, is settled by this entry).

Rationale: The board entry's title names the deliverable as the tools/ program. Its Notes make the reports the result of running that program on two named machines (Run it with --dir ... on the laptop, and with the same two on the other machine; The result goes into a measurement report ... for each machine), which only the operator can do: no agent session reaches both machines, and a report not made from those runs would carry made-up figures. The deferral moves no work out of the task's aim; it orders it after the tool exists.

## Constraints

## Auto-mode assumptions

## Open risks

## Acceptance criteria

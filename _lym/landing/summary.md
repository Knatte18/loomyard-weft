# Add tools/machinecost: comparable machine cost reports for test runs

Gates and tiers run several times slower on the XDR laptop than on the bare-metal machine,
and the measurements so far were taken by hand with different commands on each machine.
`tools/machinecost` takes the same measurements on every machine and writes them as one JSON report,
and a compare mode prints the ratio between two reports.

## What it measures

`machinecost probe` runs, in order:

- **Process starts** from Go with `os/exec`: `/bin/true`, `git --version` and `lym --help`.
  Each gets the median wall per start and the children's user and system time per start.
  The lym binary comes from `-lym` or PATH; with neither, the report lists the lym starts under `skipped`.
- **New binaries** in each `-dir` folder: a do-nothing program linked with `-ldflags=-B=0x<random>`, so its hash is new,
  run twice. The median first and second run show the hold on an unseen binary, and a folder exception's effect on it.
- **File operations** in each `-dir` folder: create, write, close and delete a small file, `stat`, `mkdir` and `rmdir`,
  and a 4 KiB write with and without fsync, each as a median.
- **A git workload** shaped like a fabric fixture, once per round: init a bare repository, clone it, add and commit a file, push.
  Each step reports its median wall, user and system time.
  One extra untimed round under `GIT_TRACE2_EVENT` counts the git processes each step starts,
  kept apart because the trace writes a file from every process.

The report also holds a machine descriptor (host, CPU model and cores, kernel, governor, Go and git versions, whether `/opt/traps` exists)
and a snapshot before and after the run: load averages, CPU pressure, mean frequency and the summed thermal throttle count.
A figure the machine does not expose is recorded as null.

`-dir` takes `name=path`; the name keys the folder's measurements, so `-dir lym=/tmp/lym-x -dir zz=/tmp/zz-x` on both machines compares row by row.
`machinecost compare base.json other.json` prints both machines and loads, then each measurement with other/base,
and warns when the two runs used different counts.

## Design decisions

- It is built with `go build -o` and run as a binary, since `go run` adds its own new-binary hold and is refused to agent sessions.
- The git workload runs under a config file of the probe's own (`GIT_CONFIG_GLOBAL`, `GIT_CONFIG_NOSYSTEM`),
  so a machine's hooks, signing or maintenance settings do not enter the comparison.
  Branch names come from git: the push is `push origin HEAD`.
- Everything is created inside a `machinecost-*` folder under each `-dir` or in the probe's own folder under the lym temp root,
  and removed before the probe returns, on failure too.
  The git workload runs in that temp folder, the same place fabric's tests make their fixtures.
- The measurement reports per machine under `docs/benchmarks/` are not part of this change:
  they need runs on the laptop and on the bare-metal machine, by the operator.

## Tests

- `tools/machinecost/report_test.go` (tier0): the median, the `-dir` flag including a name given twice,
  the `/proc/loadavg`, `/proc/pressure/cpu` and `/proc/cpuinfo` parsers, and the pairing and ratios of two reports.
- `tools/machinecost/cli_test.go` (tier0): each subcommand's exit code on good and bad arguments,
  and that a report reads back unchanged after it is written.
- `tools/machinecost/probe_test.go` (tier1): a whole probe with counts of one against two temp folders,
  pinning every measurement name, the skipped lym starts, a process count of at least one per git step,
  and that no `machinecost-*` folder is left behind.

## Merge danger

**Door:** two-way
**Blast radius:** tools

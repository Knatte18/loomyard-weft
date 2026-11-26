# Add tools/machinecost to compare test-run costs across machines

Gates and tiers run several times slower on one machine than on another, and the measurements so far were made by hand,
with different commands on each machine.
`tools/machinecost` is a new program that measures what a test run is made of and runs the same way everywhere,
so two machines, or two folders with and without an endpoint-security exception, compare on one footing.
No existing code changes.

```
go build -o <bin>/machinecost ./tools/machinecost
machinecost probe -dir lym=/tmp/lym-probe -dir zz=/tmp/zz-probe -out laptop.json
machinecost compare laptop.json desktop.json
```

The `-dir` folders must already exist.

`probe` writes a JSON report, prints it as text, and measures:

- process starts of `/bin/true`, `git --version` and `lym --help`;
- in each `-dir` folder, the first and second run of a freshly linked program with a random build ID;
- in each `-dir` folder, small-file create/write/delete, `stat`, `mkdir`/`rmdir`, and a 4 KiB write with and without fsync;
- a git workload of init, clone, add, commit and push, with times and git process count per step.

The report also holds a machine descriptor and a load snapshot from before and after the measurements.
`compare` prints each measurement of two reports with the ratio of the second to the first,
and warns when the two runs used different counts.

Closes #258

## Design decisions

- A `-dir` folder is given as `name=path`.
  The name keys its measurements, so reports compare across machines when the names match, whatever the paths.
- Run the built binary, not `go run`, which would add to the new-binary holds being measured.
- The git workload and the probe-program source live in the probe's temp folder under the lym temp root, not in a `-dir` folder.
  An endpoint exception covering that root therefore also covers the git figures.
- The git workload uses its own git config and ignores the global and system config and the caller's `GIT_*` variables.
- The probe build pins its Go environment and keeps its work folder, build cache and user config in the temp folder,
  with Go telemetry off.
  The first link of a run therefore compiles from a cold cache, outside the timed runs.
- Because `XDG_CONFIG_HOME` is overridden for the `go` commands, a version-manager shim named `go`
  that reads its config from there could fail the probe with the command's error.
  Keeping the user's value would let Go telemetry write outside the probe's folders.
- The git process counts come from one extra traced round, kept out of the timed rounds.
- An interrupt, SIGTERM or hangup still deletes every folder the probe made; SIGKILL does not.

## To check by hand

- Interrupt cleanup (`linkProbeProgram` in `tools/machinecost/newbinary.go`, and the signal handling in `main.go`):
  interrupt a probe during the new-binary part with SIGINT, SIGTERM and hangup,
  and check that no probe folder, `go` compiler or linker is left behind.
  A running `go build` is now left to finish when only `machinecost` is signalled.
  Neither the tests nor a run of this code exercises that.
- No run has listed the files written outside the probe's folders since the build cache and telemetry were moved inside.
- The per-machine reports under `docs/benchmarks/`, beside `gate-test-xdr.md`, are not part of this change.
  Run the built binary on the laptop with Cortex XDR and on the other machine,
  each with one `-dir` under `/tmp/lym-*` and one under `/tmp/zz-*`, and write the reports from those runs.

## Merge danger

**Door:** two-way
**Blast radius:** machinecost

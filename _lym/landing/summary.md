# Research: time the fixed steps of a tier1 test call

An earlier study found that every tier1 test call pays a fixed cost before any test runs, and did not split it by step.
This change adds the report that times the steps one by one, with its per-call data and an entry in the research index.
It changes no code.

Report: `docs/research/speed/gate-fixed-cost-2026-11-27/report.md`, data in `calls.tsv` beside it.

Main results, from runs on an unchanged tree with a warm build cache:

- `go build ./...` takes 3.0 to 4.4 s. Read from Go's source, it links every command of the module and discards the binaries.
- The build of the `lym` binary takes 2.4 to 3.2 s on every call, whether or not a test uses the binary.
- `go vet` of one or ten packages takes under 0.3 s, and `go vet ./...` 1.2 to 1.6 s.
- `-ldflags=-w` has no effect, since `go test` already links test binaries with `-s -w`.

The report recommends replacing `go build ./... && go vet {targets}` with `go vet ./...`,
and building `lym` only when a targeted package's tests need it.
Neither is made here.

Limits of the measurement:

- The one-package and ten-package cases ran three times, the whole tier1 set twice.
- The skip-build figure is derived by subtracting the build's time from the call's, not run on its own.
- The steps that only a `--gate` call has were not timed.
- `go build ./...` and `go vet` were each run alone through two temporary manual tiers of the hub config,
  which are not part of this change.

## Merge danger

**Door:** two-way
**Blast radius:** docs

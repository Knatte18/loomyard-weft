# Measure the new binaries a gate run builds and launches

Adds `docs/research/gate-fewer-launches-2026-10-31/report.md`, the measurement the board entry asks for; no code, config or gate tier changes.

## What the report holds

- The count of test binaries new to the Go test cache per tier, from two gate runs on Neve: a two-package `--upto 2` run and a whole-suite `--upto 2 --all` run.
- The cold-cache upper bound per tier and the per-package import fan-in, derived from `go list` over the module.
- Candidate cuts ranked by new binaries saved, each with its risk to result equality, and the OSL-1046 gain at about 10 s per new binary.

## Design decisions

- The `--upto 2` run took explicit targets, because the clean branch had an empty diff and a diff scope would have run no tier.
- Counts the cap of two runs could not observe are derived from the package graph and marked as derived in the report.
- What `go build ./...` links was not observed, since a raw module-wide go run is refused; the report says so.

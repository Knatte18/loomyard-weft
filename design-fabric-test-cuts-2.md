# Second fabricengine test cut, from the package-scoped redundancy tool

`fabric-test-cuts-2` · Note · enhancement, fabric, testkit

## Seen (orch, 2026-11-08)

The first cut (`f2ac2bf7`) dropped 6 `internal/fabricengine` tests and added 8 tier 0 tests: 760 tests before, 762 after, and no time measured.
Its body told the writer to confirm each drop against the module-wide `go run ./cmd/testtiming -redundancy` check,
which counts the fixture packages' blocks too and lists almost no fabricengine test as contained, so it vetoed nearly every drop.
The package-scoped study (`docs/research/speed/fabric-test-redundancy-2026-11-08/`) found 267 unmarked tests that can go together.

## Idea (operator, 2026-11-08: build the map first)

A second cut runs once the redundancy tool from `redundant-test-finder` has landed with package-scoped coverage.
It drops from that tool's removable set for `internal/fabricengine`, not from the module-wide check.
Each test is still read before it goes; one guarding data is kept with `//testtiming:keep <reason>`.
Slowest first; the writer delegates batches by file to subagents, never two on one file.
The package's tier 1 time in a gate pass is measured before and after, and both numbers stand in the PR.

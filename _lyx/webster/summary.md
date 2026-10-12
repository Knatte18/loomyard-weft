# Single seed parser: report reads seeds through ParseSeed

Batch 16 (`single-seed-parser`) was the only batch and recorded `done` at `1a0e9e54c0bb4d916175ed6b99f0880309eaff6b`, with no reported deviations.

The implementer fork deleted `ParseSeedTolerant` from `internal/shedrun/seed.go`.
`readLoomRecords` in `internal/reportengine/runs.go` now decodes a run's `seed.json` through `shedrun.ParseSeed`, and its doc comment no longer describes the retired function.
`TestParseSeed` dropped its `ParseSeedTolerant` assertions.
`TestCompute_SelectsRuns` gained a `badseed` row whose seed names an unknown driver and must come out as a partial run with a finding.

The fork reported the card and batch gates green for `./internal/shedrun` and `./internal/reportengine`, including the integration tier, plus `lyx loom lint-comments` and `lyx loom lint-pattern`.
The plan-level verify gate runs after this turn ends.

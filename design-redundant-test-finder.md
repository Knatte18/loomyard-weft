# A mechanical check flags a test whose package-scoped coverage other tests already cover

`redundant-test-finder` · Note · enhancement, testkit, gate

## Seen (orch, 2026-11-08)

`PATTERN-test-speed` forbids a test that covers no behaviour another test does not, but only review enforces it.
Its tool, `go run ./cmd/testtiming -redundancy`, measures coverage over the whole module, so a test's blocks include the fixture packages' blocks; it listed no candidate in `internal/fabricengine`.
The coverage study `docs/research/speed/fabric-test-redundancy-2026-11-08/` scoped coverage to the package under test and found 370 of 590 tier 1 tests contained.
Its scripts (`coverage-runner.sh`, `analyze-coverage.sh`) run each top-level test once on its own and compare the covered blocks.

## Idea

Make the study's method a mechanical check in lym:

- `cmd/testtiming -redundancy` scopes coverage to the package under test (`-coverpkg` naming only that package), as the study did, and keeps the greedy removable set.
- A stored coverage map per package, refreshed when the package's tests change, so a check needs to run only the new or changed tests.
- A gate or review step that flags a test a diff adds or changes whose package-scoped blocks are all covered by other tests, naming the covering tests.
  It is a finding for review, never an automatic refusal or deletion: block coverage does not see assertions.
  A test kept with `//testtiming:keep <reason>` is not flagged.

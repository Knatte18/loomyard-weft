# Coverage map records 0 seconds for nearly every test

`map-test-seconds-zero` · Note · bug

Found by the map-tooling-trial.

In the `internal/boardengine` map from `lym gate test --wait --map`, 132 of 136 tests record `seconds: 0` and the slowest records 0.29.
`lym gate redundant internal/boardengine` then lists every one of its 14 candidates with `seconds: 0`.
Its "greedy pass, slowest first" has no order to work from, and the standalone seconds it reports say nothing about what deleting a test saves.
Record each test's time at a resolution that separates tests, or say in the report that the figure is a floor.

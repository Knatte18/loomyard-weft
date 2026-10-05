# Prune the remaining test packages and guard the budget

`test-suite-prune-rest` · Task · enhancement

Depends on: [`test-suite-prune-core`](design-test-suite-prune-core.md)

Stage 2c of getting the test suite under control: prune every package `test-suite-prune-core` left, record the after-state and guard it.

## Clean

In every package outside `test-suite-prune-core`'s set, the same rules as there:

- Delete the tests the tool reports as redundant, after checking each one does not pin a contract the coverage view misses (an error message, an output shape).
- Fold near-identical tests into table-driven tests.
- Move integration tests that need neither git, tmux nor processes to the untagged tier with `internal/testkit` fakes.

## Budget guard

A guard test pins a per-package budget for test count and wall time at the after-state; exceeding it fails until the budget is raised deliberately in the same commit.

## Done when

- The after-state is recorded beside `test-suite-measure`'s before-state in `docs/benchmarks/test-suite-timing.md`.
- The whole suite's test count and the integration tier's wall time have dropped substantially; the PR states both numbers.
- Coverage of non-test code has not dropped beyond what deleted redundant tests explain.

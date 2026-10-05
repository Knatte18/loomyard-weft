# Prune the remaining test packages

`test-suite-prune-rest` · Task · enhancement

Depends on: [`test-suite-measure`](design-test-suite-measure.md)

Stage 2c of getting the test suite under control: prune every package outside `test-suite-prune-core`'s set.
It runs in parallel with `test-suite-prune-core`, after `test-suite-measure`, using its coverage tool, its candidate report and the consolidated Testing rules.
The hub coordinates the two runs (shared files such as `internal/testkit` belong to `test-suite-prune-core`); the after-state and the budget guard follow in `test-tiers`.

## Clean

In every package outside `internal/fabricengine`, `internal/websterengine`, `internal/reedengine`, `internal/shuttleengine`, `internal/loomcli` and `internal/shedadapters`:

- Delete the tests the tool reports as redundant, after checking each one does not pin a contract the coverage view misses (an error message, an output shape).
- Fold near-identical tests into table-driven tests.
- Move integration tests that need neither git, tmux nor processes to the untagged tier with `internal/testkit` fakes.

## Done when

- The tool, rerun on these packages, reports no redundant candidate left without a stated reason.
- The PR states the test count and wall time per package before and after.
- Coverage of non-test code in these packages has not dropped beyond what deleted redundant tests explain.

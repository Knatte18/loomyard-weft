# Status strand chosen by the run's driver

All three batches reported `done`, and the plan-level `## verify:` integration run reported `status: OK` at HEAD `dfc07967`.

- **01 status-strand-removal-helper** (`eb903812`): added `removeStatusStrands` to `internal/loomcli/bootstrap.go`, which removes every `loom-status` strand non-recursively and warns on failure, plus the table-driven `TestRemoveStatusStrands`.
- **02 step-drops-status-strand** (`e9009971`): `loomPreStep` in `internal/loomcli/arm.go` now calls reed `Up` in place of `ensureStatusStrand`.
  Added the new smoke file `internal/loomcli/smoke_drivingsurface_test.go`, whose tests pin that `step` adds no status strand, leaves an existing one alone, and refuses when reed `Up` fails.
  Comments in `sharedbootstrap.go` and `internal/shedcli/parity_test.go` were updated to match.
- **03 start-strand-by-driver** (`dfc07967`): `start` runs reed `Up` first, then keeps the status strand on a go-driven run or removes every one on an llm-driven run.
  `ensureStatusStrand` no longer calls `Up`. The `Long` help text, `CONSTRAINTS.md` and `docs/overview.md` were updated.
  New smoke tests cover the llm removal and a failed `Up` on either driver.
  Existing smoke tests were adjusted, and four extra `seedGoDriverRun` calls were added because unseeded fixtures now count as llm.

Deviations: none reported as deviations. `record-batch` gave informational scope-outside-plan warnings for the touched test symbols, which match the reported work.

The batch 03 fork noted that two bootstrap smoke tests (`MalformedStatusProceedsToHandoverAndLogsWhy`, `DiedDriverProceedsToHandoverAndLogsWhy`) fail when `TMUX` is set in the environment.
It did not check whether they also fail on the parent commit.

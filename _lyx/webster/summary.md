# loom start refuses a mid-merge pair

All three batches finished `done`, and the integration suite reported `status: OK`.

- **01 merge-in-progress-kind** (`066dc898f`): added `StartMergeInProgressKind = "merge_in_progress"` to `internal/shedrun/paths.go`, beside `StartNotParkedKind`.
- **02 midmerge-probe** (`8865b8a65`): added `fabricengine.MidMerge` in `midmerge.go` with integration tests for each state.
  It also refactored the four git probes in `mergestate.go` into one shared reader, which `foreignMergeStatePresent` now wraps, and updated `doc.go`.
  Both are informational scope-outside-plan notes only.
- **03 start-refuses-mid-merge** (`26185e211`): `start` now probes only when it would spawn a driver or resume a parked one.
  It refuses with `merge_in_progress` and the conflicted paths, or with a plain error if the probe fails.
  It added a `midMerge` seam on `loomCLI`, updated the `startCmd` help text and `docs/overview.md`, and added `start_midmerge_test.go`.
  Its deviations are informational only.

The plan-level `## verify:` ran once at `26185e211` and passed.

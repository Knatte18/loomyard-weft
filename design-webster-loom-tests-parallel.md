# Run webstercli and loomcli tier1 tests in parallel and share read-only fixtures

`webster-loom-tests-parallel` · Task · enhancement, webster, loom, testkit

## Aim

`docs/research/speed/test-time-measure-2026-11-27/report.md` measured `internal/webstercli` at 15 to 17 s and `internal/loomcli` at 18 to 21 s of tier1 test time.

## Work

- `internal/webstercli`: its tests run one after another because they set the environment or working directory, which bars `t.Parallel`.
  Pass those values in instead, then mark the tests parallel. `TestResetCmd` (6.8 s) is the longest.
- `internal/loomcli`: share one built hub copy between the tests that only read it, and fold rows of the same shape.
- Keep every assertion; drop none.
- Measure each package's tier1 time before and after, twice each, and put both in the change description.

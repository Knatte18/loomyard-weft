---
round: 2
focus:
  - "In `internal/gatecli/prebuild.go` (`needsPrebuiltLym`, `invocationsUseLymBinary`) and `internal/gatecli/tiers.go` (`scopeInvocations`): for every command shape and request shape, including a tier command with neither `{targets}` nor `{test_targets}` run under explicit targets, does the build decision follow the set of tests the command runs?"
  - "Across the callers of the build decision, the plain run path and the `--copies` path in `internal/gatecli/copies.go`: does a test fail if either path stops honouring the decision, in the build direction and in the skip direction?"
  - "In the comments in `internal/gateslot/tiers.go` and `internal/gateslot/pool.go`, `pattern/PATTERN-test-speed.md`, `docs/benchmarks/running-tests.md` and the `gatecli` package doc: does each statement of when `lym` is prebuilt match the code?"
---

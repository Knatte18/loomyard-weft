---
round: 1
focus:
  - "Safety conditions: for deleted production functions, check that none is reached on windows or darwin (build-tagged files, e.g. internal/fslink junctions, reedengine process-tree code) or by reflection, go:linkname, a template function map or a lookup by name."
  - "Deletions with tests: check that each deleted function's tests tested only it, and that no deleted test covered behavior of a live function that now has no coverage."
  - "Moves into _test.go or export_test.go: check that the moved code is unchanged in behavior and that no production file still needs it, including builds under other GOOS values."
  - "RunCLI/RunCLIIn removal: check that no wrapper or reference to one is left, that every replacement call passes the same arguments and the same stdout/stderr, and that configcli's production call of the former fabriccli.RunCLIIn behaves as before."
  - "cmd/lym run and main fold: check that one function now serves both, that exit codes and the separate stdout/stderr writers are preserved, and that no test-only branch remains in it."
  - "Guard allowlist in cmd/lym/testonlycode_linux_test.go: check each entry's Why against real use by another package's tests, and that no entry could instead be a _test.go helper or a testkit member under the pattern's own rule."
  - "PATTERN-test-only-code.md and the edits to PATTERN-cli-cobra.md and PATTERN-shed-driver.md: check that the rule matches the goal's rule, and that the guard enforces what the pattern says, including the stated bounds."
  - "Test economy: judge the guard test and any moved or new tests against the redundancy report; the TestExitCode_* and TestJSONHelp_Schema tests the report lists as contained need their assertions compared with TestRunDispatchesToUninitializedRepoModules."
  - "Change description: check the claims in _lym/landing/summary.md, including which kept functions it names as references or oracles, against the diff."
---

Seed focus drawn from the decision record's four items and four safety conditions, and from the diff's commit messages.

# No test is skipped by design

`test-skips-os-tags` · Task · enhancement, testkit, reed, loom, fabric, gate · running · Finalize

## Seen (operator, 2026-10-26, OSL-1046)

The adopt check's gate run on Linux reported skipped tests in tier0 and tier1.
`TestUp_BootValidation/TmuxConfUnvalidatedOnWindows` (`internal/reedengine/lifecycle_test.go`) skips itself on every machine but Windows, so a Linux run counts it as a skip; tier1 has more Windows-only tests of the same kind.
`TestBuildLoomShed_OutputShape` (`internal/loomcli/sharedbootstrap_test.go`) skips itself whenever `buildLoomShed` fails, which it does because the test sets up no real fabric, so it most likely tests nothing on any machine.
Tier1 also skips the deliberate `TestCrossProduct` matrix cells (`internal/fabricengine/livestate_matrix_test.go`) and the child-process helper tests, which skip unless a parent test spawns them.

## Decided (operator, 2026-10-26)

No test is skipped by design: a test that is meant to skip has no place in the suite.

- A test that only means something on one OS moves into a file behind a `//go:build` tag for that OS, so other machines never compile it.
- `TestBuildLoomShed_OutputShape` either builds the real fabric it needs and fails when `buildLoomShed` fails, or is deleted.
- The matrix cells that are skipped by design are not generated at all.
- The child-process helpers stop being test functions the runner sees, for example by dispatching from `TestMain` on an environment variable.
- A skip that remains is a missing prerequisite, such as `TestGateTest_DotnetFixture` without its NuGet packages, so a skip in the gate log is always worth reading.
- `TestClaude/neither_skips/inner` in `internal/testkit/llmkit` is the skip helper's own test and stays.

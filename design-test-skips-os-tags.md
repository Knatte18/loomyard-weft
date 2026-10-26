# OS-only tests behind build tags, and no test that skips on its own failed setup

`test-skips-os-tags` · Note · enhancement, testkit, reed, loom

## Seen (operator, 2026-10-26, OSL-1046)

The adopt check's tier0 run on Linux reported three skipped tests.
`TestUp_BootValidation/TmuxConfUnvalidatedOnWindows` (`internal/reedengine/lifecycle_test.go`) skips itself on every machine but Windows, so a Linux run counts it as a skip.
`TestBuildLoomShed_OutputShape` (`internal/loomcli/sharedbootstrap_test.go`) skips itself whenever `buildLoomShed` fails, which it does because the test sets up no real fabric, so it most likely tests nothing on any machine.
The third, `TestClaude/neither_skips/inner` in `internal/testkit/llmkit`, is the skip helper's own test and stays.

## Idea

- A test that only means something on one OS moves into a file behind a `//go:build` tag for that OS, so other machines never compile it and a skip in the gate log is always worth reading.
- `TestBuildLoomShed_OutputShape` either builds the real fabric it needs and fails when `buildLoomShed` fails, or is deleted.
- The same sweep covers the tier1 skips of the same kinds.

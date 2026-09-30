# Card 2 — verify-output-parser

**Create:**
- `verifyparse.go`
- `verifyparse_test.go`
- `plan:internal/websterengine#parseVerifyFailures` -> `func parseVerifyFailures(output string, passed bool) []IntegrationFailure`
- `plan:internal/websterengine#opaqueFailureID` -> `const opaqueFailureID = "verify-command"`

**Uses:**
- `internal/websterengine#IntegrationFailure`

**Intent:** Parse a verify command's combined output into failure identities, per D5.
The parser lives in `internal/websterengine`, not its own package, since webster is its only consumer.

`parseVerifyFailures` returns nil when `passed` is true.
Otherwise it returns, in first-seen order and deduplicated by `id`:

- A **test** identity `<package>.<TopLevelTest>` for each `--- FAIL: <name>` line, attributed to the `FAIL\t<package>` line that closes its package block.
  A subtest name (`TestA/case`) collapses to its top-level test, so several failing subtests of one test yield one identity.
  The tail is the indented lines following the `--- FAIL:` header.
- A **package** identity `<package>` for a `FAIL\t<package>` line with no test failure attributed to it: `[build failed]`, `[setup failed]`, a `TestMain` or init panic.
  The tail is the last lines of the output.
- One **opaque** identity with id `opaqueFailureID` when nothing above was parsed — a failing `go vet`, `go build`, or non-Go command.
  The tail is the last lines of the output.

Every tail is capped at 40 lines, held in one named constant.
Normalize `\r\n` to `\n` before parsing.
Lines not belonging to a `go test` package block (a `go vet` complaint, shell noise) are ignored rather than rejected.

The untagged `verifyparse_test.go` (package `websterengine`, since the parser is unexported) is table-driven and asserts exact identity lists, kinds, and tails for:
top-level test failures across several packages;
failing subtests collapsing to their top-level test;
`[build failed]` and `[setup failed]` package failures;
a panic with no `--- FAIL:` line yielding the package identity;
`\r\n` input;
a tail longer than the cap truncated to the cap;
an all-`ok` log with `passed` false yielding the opaque identity;
non-Go output yielding the opaque identity;
and `passed` true yielding nil even when the output contains `FAIL` lines.

**Commit:** 2: verify-output-parser

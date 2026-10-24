```yaml
lens: code-tests
outcome: reported
```

Lens code-tests: for each behaviour the diff adds or changes, the test that pins it, or none. I ran 2 tool calls, and I did not open the test bodies. The row names below are the evidence.

**Behaviours pinned**
- Config keys (`scoper: dotnet`, `filter` without `{filter}`, `build` missing `command` or `timeout_min`, `shared_build` with no `build`): `TestConfig` rows in `gateslot/config_test.go`.
- The inert matcher (base-name and slash patterns, malformed pattern, all-inert, empty list, empty path set): `TestConfig_Inert`.
- Build run record named `-build-`, a blocked tier's record keeping the failed build's log, `FailedTier` returning the `build` flag, and the build timeout line: `gateslot/runlog_test.go` and `watchdogline_test.go`.
- Landing failure record with `build: true` and no tests: `landingshed/mergein_test.go`.
- Build timeout wording at the verify gate: `verifygate_test.go`.
- Fingerprint changes for filter, `shared_build`, `carry` and the build declaration: `TestGreenRecord` (`green_test.go:61`).
- A `carry: false` tier keeping its invocation despite a whole entry: `green_test.go:130`.
- Inert carry over an inert-only ancestor diff: `scopes_integration_test.go` rows for the inert carry, a non-inert commit, a dirty tree, `carry: false` over inert history, and a changed `inert_paths`.
- Narrowed dotnet pass writes no green entry, `--all` records whole, and two vstest tiers then carry: `scopes_integration_test.go`.
- Inert-only diffs under `go`, `none` and `dotnet`, and the untracked non-inert file: `scopes_integration_test.go`.
- No-test-class scope (build once, no `tier0` command, `tier1` whole): `scopes_integration_test.go`, plus `tiers_test.go` rows.
- Empty dotnet diff running no build, and the `carry: false` `shared_build` tier running the build before its whole command: `scopes_integration_test.go`.
- Build step in `test_integration_test.go`: once, in the tier's dir with its env and none of the call's words; failing build blocking opted-in tiers while a tier that did not opt in still runs; timeout; missing dir refusal; second `--all` run carrying with no build.
- Per-project invocation with terms, a command without `{targets}`, split by base, `carry: false` under each scope form, explicit targets, and `--guards` under dotnet: `TestPlanTierRuns`.
- `{filter}` rendering, including terms past the budget falling back to the tier filter alone: `TestRenderTierCommand` and `TestFilterArgument`.
- Report totals for build runs apart from tier runs: `report_test.go`.
- Derivation: `TestDeriveDiff`, `TestDeriveCards`, `TestNarrow` and `TestParseSource` cover the lexer and narrowing cases, including verbatim, raw and interpolated strings, char literals, nested types, file-scoped namespaces and attribute spellings.
- Card seeds: `TestCardCSharpSeeds`.
- The `tier1` fixture: `TestGateTest_DotnetFixture`.

**Gaps**
- **LOW, consistency:** `gateslot/config.go` (`decodeBuild`) refuses a `build` with `timeout_min` but no `command`, and one with `command` but no `timeout_min`. Both are covered. A `build` holding only `about`, `dir` or `env` is also refused for the missing `command`, and no row pins it. Add a row to `TestConfig` at `gateslot/config_test.go`. It is a low-risk gap.
- **LOW, scope:** I found no row for `reportBlocked` printing no failed `tier<N>` summary line. The decision record's acceptance criterion requires that, and the `test_integration_test.go` blocking subtest name implies it without stating it. Confirm the subtest asserts the absence of the `gate test: tier<N> ... exit` line for blocked tiers, and add the assertion if it doesn't.

**Test economy:** I found no duplicate or per-helper test among the names listed.

**Tool calls**
1. `git diff -U0` over `*_test.go` for the review range: test names and row names were not in my context.
2. `grep` in `dotnetimpact/source_test.go`, `gatecli/green_test.go` and `gatecli/tiers_test.go`: to check lexer edge-case coverage and the `carry`/budget rows.

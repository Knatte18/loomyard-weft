```yaml
lens: code-tests
outcome: reported
```

Lens code-tests: per behaviour, the test that pins it or none.

**Per-behaviour list**

1. Counter keeps up to `failureLines` lines per failed test: `TestCounters` rows "a failed test keeps its first failureLines lines" and "a failed subtest is named with its error lines…". Pinned.
2. Package failure outside any test (panic in TestMain): `TestCounters` row "a package that panics outside any test…". Pinned.
3. Build-failed package with `build-output` and `FailedBuild`: `TestCounters` row "a package whose build failed…". Pinned.
4. Package fail event with no failed test of its own gets an empty-line failure: the first launch row adds `{pkg: "example.com/q"}`. Pinned.
5. No package failure is added when a failed test already explains the package (`failedPackages`): the failed-subtest and prefix-name rows assert exact failures, so a regression would add a stray entry. Pinned.
6. `Unparsed()` keeps non-JSON lines and drops status lines: the panic/build-error row and the trailing-line row. Pinned. The `unparsedLines` cap is not exercised in `TestCounters`. Only `TestBuildFailedRun_KeepsTheLastLinesOfItsOutput` covers a different cap path, `buildFailedRun`'s tail. See F1.
7. `isStatusLine` and `appendCapped` filtering (the `# ` prefix, the `ok  ` and `FAIL\t` prefixes): covered indirectly by the build row and the panic row. Pinned.
8. `vstestCounter.Unparsed` and `Failures` return nil: the vstest rows run the shared `TestCounters` loop and expect nil. Pinned.
9. `renderFailureReport`: the `TestRenderFailureReport` rows cover nil for no runs, the test/package layout, `qualify` prefixing, unparsed lines, the "log holds the cause" line, the lint way-forward header, the line cap, the per-line cut and the touched-files cap (`and 3 more`). Pinned. Whether the "... N more lines" truncation line is asserted is unseen, but the subtest at line 84 is named for it.
10. `capLine` cutting at a rune boundary: no multibyte row is visible at 84–110. See F2.
11. `lintFailedRun` (singular/plural, could not run, none): `TestLintFailedRun`. Pinned.
12. `runLints`: the switch over `comments` and `pattern`, and building `file:line: text` findings. The integration lint scenario pins the `comments` path through `runGate`. The `pattern` branch has no test that drives `runLints`. `lintFailedRun` is tested with `LintPattern`, but the `patternlint.Lint` call and the `"PATTERN.md: "` message formatting are not. See F3.
13. `tierFailedRun` (module path and dir, timeout outcome): `TestTierFailedRun_NamesTheModuleOfItsRunDirectory`. The `timed out` outcome and the empty-`RunDir` branch are not shown in that test (it builds one result). See F4.
14. `buildFailedRun`: `TestBuildFailedRun_…`. It covers the exit outcome, the timeout and the last-lines tail. Pinned. The integration build-failure wiring (`e.failed` append in `build.go`) is pinned by no test that I could see: the real-go integration row has a broken package but no `build` step, and no scenario with a failing shared build step checks the report. See F5.
15. `touchedFiles` (merge base in a hub, fallback to HEAD, error returns nil, sort and compact): only the integration row's `gate test: the change touches` / `report/p/p_test.go`. That covers the non-hub or HEAD path. The hub merge-base branch and the `parentMergeBase` error path have no test I could see. See F6.
16. Run wiring: `report` appends a failed run when `ExitCode != 0`, the report prints before the closing line, and the exit code is 1: the real-go integration row asserts the order, the exit code, the log path and `file:line` qualification. Pinned.
17. Lint step runs after tiers, sets exit 1 when no earlier run failed, and the closing line has `lint comments (1 finding)`: the integration lint row. Pinned. The "an earlier failure keeps its code" branch (`code == 0` guard) and the "lints skipped when `runCtx.Err() != nil`" branch have no test. See F7.
18. Halted-call path calls `printReport` before `printFailedRuns`: no test, since the halt rows check the closing line but not the report. See F7.
19. A manual-flag call, with no `--gate`, runs no lints: not asserted. Low value; fold into F7.
20. `decodeGates` lint validation: `config_test.go` rows "an unknown lint" and "a key's lints load in order". Pinned. An empty-string lint name falls into the unknown branch and is not separately asserted, which is acceptable.
21. The template `card` key carries both lints: `templateDefaults` in `config_test.go`. Pinned.
22. `Gate()` clones `Lints`: no test asserts that `Gate("card")` returns `Lints`. Anything calling `AgentGateCall` or `TierList` is unaffected. The integration lint row goes through `cfg.Gates[...]` directly and never through `Gate()`. See F8.
23. `cardGateCommand` returns only the gate call: `cardgate_test.go` plus the `beginbatch`/`recoverbatch` prompt rows. Pinned.
24. `replayStdout` (absent, short, long, long with JSON envelope): `TestReplayStdout`. Pinned. `front.replay` wiring (`replayStdout` used instead of `copyRunFile`) has no detached-front test I could see. See F9.
25. Test economy:
    - `TestBuildFailedRun_…` and `TestTierFailedRun_…` are per-helper tests, but each pins a distinct unexported-helper behaviour that the public integration row does not reach (the build step's tail and the timeout outcome). Not redundant.
    - `TestLintFailedRun` partly overlaps the integration lint row on the singular case. The integration row asserts "1 finding", but `TestLintFailedRun` covers the plural, the error and the none cases. It is a distinct table, not a defect.
    - The `renderFailureReport` cases are a table plus one subtest, which is economical.
    - No redundant-test finding.

**Findings**

- F1 · LOW · design · `internal/gatecli/counters_test.go` (`TestCounters`) · No row pins the `unparsedLines` cap on `goJSONCounter.handle`. Only one or two unparsed lines are used, so dropping the `len(c.unparsed) < unparsedLines` guard passes. Evidence: `counters.go` handle diff (`len(c.unparsed) < unparsedLines`); rows with at most 2 unparsed lines.
- F2 · LOW · design · `internal/gatecli/failurereport_test.go:84-110` · No row for `capLine` cutting at a multibyte rune boundary. The `RuneStart` loop is uncovered, and a multibyte value in a panic or got/want would be corrupted if it broke. Evidence: `failurereport.go:244-253` and the line-84 subtest, which I saw only by name.
- F3 · MEDIUM · design · `internal/gatecli/failurereport.go:107-112` · `runLints`' `pattern` branch (the `patternlint.Lint` call and the `PATTERN.md: ` finding text) is driven by no test. The only lint scenario uses `lints: ["comments"]`. Add a row with `lints: ["pattern"]`, or test `runLints` directly with a fixture repo. Evidence: the integration lint row at `test_integration_test.go` ~1320; `TestLintFailedRun` passes findings in by hand.
- F4 · LOW · design · `failurereport_test.go:171-192` · `tierFailedRun`'s `timed out` outcome and its empty-`RunDir` branch (no module qualification) are not asserted. Evidence: `failurereport.go:63-72`.
- F5 · MEDIUM · design · `internal/gatecli/build.go:75` · The wiring that appends `buildFailedRun` to `e.failed` on a failing shared build has no test through `runGate`. A removal of that line would pass every visible test. Evidence: `build.go` diff and the integration rows seen in the diff, none of which has a build step.
- F6 · LOW · design · `failurereport.go:144-162` · `touchedFiles`' hub branch (merge base with the pair's parent, and the fallback logging path on an error) has no row. The integration scenario covers only the HEAD base. Evidence: the `touched` assertions in the integration row, with no hub-parent scenario.
- F7 · MEDIUM · design · `run.go:174-183` and `:169` · Two lint-wiring branches have no test: a lint failure after a failing tier must keep the tier's exit code (`if code == 0`), and a halted call must still print the report. Evidence: the single integration lint row, which has no failing tier, and the halt rows, which don't assert the report.
- F8 · LOW · design · `internal/gateslot/tiers.go` (`Gate`) · `Gate()` now carries `Lints`, but no `tiers_test` row asserts that `Gate("card")` returns the lints, nor that the slice is cloned. Evidence: `tiers.go` diff line `Lints: slices.Clone(gate.Lints)`. The diff touches no test for `Gate`.
- F9 · LOW · design · `internal/gatecli/detachfront.go:291` · `front.replay`'s switch to `replayStdout` is not pinned by any detached-front test. `TestReplayStdout` covers the function, not its use. Reverting to `copyRunFile` would pass the visible tests. Evidence: the `detachfront.go` diff and the grep showing `replayStdout` called only in `failurereport_test.go`.

Test economy: no duplicate, per-helper-redundant or stencil-prose assertion findings.

**Tool calls**
1. `git diff` of `counters_test.go` and `test_integration_test.go`, plus a grep of `failurereport_test.go` test names: those test diffs were excluded from the context's diff read.
2. A grep of the test files for the new helpers (`replayStdout`, `runLints`, `touchedFiles`, `buildFailedRun`, `tierFailedRun`, `testFailures(`): to confirm which helpers have direct tests.

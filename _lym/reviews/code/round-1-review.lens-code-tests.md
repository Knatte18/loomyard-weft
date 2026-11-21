```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens of this round. For each behaviour the diff adds or changes, I looked for the test row that would fail if the behaviour broke.

**Coverage list, per behaviour**

1. **Results-directory args added for a vstest tier whose command holds `{args}`:** pinned. Every row of `TestRunTierReadsTheVSTestResultsFile` finds `--results-directory` among the args (`tierrun_test.go`, the `command` built at diff line 367).
2. **A command without `{args}` keeps the console counts:** pinned indirectly. The vstest rows of `TestRunTierNoTestNotice` use a command with no `{args}` (`tierrun_test.go:385`) and assert the console-derived result.
3. **TRX counts replace console counts (detailed logger case):** pinned by row `ADetailedConsoleLoggerLeavesTheCountsToTheResultsFile`.
4. **A failed test, with its message and stack lines, is read from the TRX:** pinned by row `AFailedTestIsReadFromTheResultsFile`.
5. **Outcome mapping (Error, Timeout and Aborted count as failed; other outcomes count as skipped):** none. Only `Passed` and `Failed` appear in the fixtures.
6. **No readable TRX, or an unparsable file, leaves the console counts in place:** none.
7. **Several invocations keep earlier invocations' counts (the `startInvocation` baseline):** none. Every row uses one invocation.
8. **A crash run message is printed, logged and added to the causes on a failed run:** pinned by row `ACrashedTestHostNamesItsCause`.
9. **Run messages are not printed when the run passes:** none.
10. **The results directory is removed after the run:** pinned on the normal path by the stat check in every row. Removal on the render-error path has no row. The `tierrun.go:570` mutation survivor is the log-only branch, so no finding there.
11. **`envOverrideLine` content (differing value named; shared or unset values not named):** pinned by `TestRunTierNamesEnvThatDiffersFromTheSession`, both rows.
12. **`runTest` fills `request.sessionEnv` from `os.Environ()`:** none.
13. **`Config.Gate`'s unknown-key refusal lists the keys in sorted order, or says it holds none:** pinned by `TestConfig_TierUse` (`config_test.go:11-17` of the diff).
14. **`MeasureCall` (key with targets, key without targets, missing key, undeclared tier, no tiers):** pinned by the `TestConfig_TierUse/MeasureCall` rows. Two cases have no row: a key mixing a tier that takes targets with one that does not, and a fallback where tier0 takes no targets.
15. **`DarnTable` refuses an empty `MeasureCall`:** pinned by `TestDarnSpec_Refusals`, row `empty MeasureCall`.
16. **The gated fragment renders the call, the ungated one omits it, and the hard-coded `--gate darn-write` is gone:** pinned by `TestDarnSpec`, gated and ungated rows.
17. **The darn wiring renders the measuring call from the hub's `gate.yaml`, with an absent config treated as zero:** the absent path is pinned implicitly, because the Preflight test builds the table (`wiring_darn_test.go:138-141`). That the call follows the config has no row.
18. **The marker set includes `measure_call`:** pinned by `TestStencils_TopLevelMarkers`.

**Findings**

- **MEDIUM · design · `internal/gatecli/counters.go:475-476`:** no test has two vstest invocations in one tier run. If `startInvocation` stops recording the baseline, the second invocation's `replaceInvocation` drops the first invocation's TRX counts and failures, and no test notices. This is the mutation report's two surviving mutants. The fix is a two-invocation row in `TestRunTierReadsTheVSTestResultsFile` asserting the summed counts. Evidence: every row uses `invocations: []invocation{{dir: dir}}` (diff piece 004, line 370).
- **LOW · design · `internal/gatecli/trx.go:565-569`:** the outcome mapping that `doc.go` states has no row: Error, Timeout and Aborted count as failed, and any other outcome as skipped. The fixtures hold only `Passed` and `Failed`, and the record's `Skipped` is asserted but is always 0 (tierrun_test diff lines 322-325, 379).
- **LOW · design · `internal/gatecli/trx.go:521-543`, `tierrun.go:543-547`:** the documented fallback is unpinned. An invocation that writes no TRX, or writes an unparsable one, should keep its console counts. Every row copies a valid fixture, so breaking the `read` guard, for example by always calling `replaceInvocation`, would pass.
- **LOW · design · `internal/gatecli/run.go:121`:** nothing pins `request.sessionEnv = os.Environ()`. `TestRunTierNamesEnvThatDiffersFromTheSession` sets `sessionEnv` directly (`tierrun_test.go:582`), and the integration tests now strip `GOFLAGS` so no env line appears. Deleting line 121 turns the whole env-line feature off on the CLI path, and every test stays green.
- **LOW · design · `internal/gateslot/tiers.go:445`:** two `MeasureCall` semantics are unpinned. One is "a target word when *any* of the key's tiers takes targets": no row has a key whose tiers mix one that takes targets with one that does not. The other is a fallback where tier0 takes no targets: the `nope` row's tier0 holds `{targets}`. Switching `ContainsFunc` to an all-tiers check, or always adding the target word in the fallback, passes every row.
- **LOW · design · `internal/loomcli/darnwiring.go:317-334`:** no test asserts that the darn prompt's measuring call follows the hub's `gate.yaml`. `TestDarnSpec` feeds a constant `MeasureCall`, and the wiring tests only build the table. Replacing `gateCfg.MeasureCall(darnWriteGateKey)` with any non-empty literal passes. The error return for a malformed config other than an absent one has no row either.
- **NIT · design · `internal/gatecli/tierrun.go:152`:** no row has a passing run whose TRX carries a run message. Nothing checks that a passing run prints no run message and adds no cause.

**Tool calls**

1. `grep` of the loomcli wiring tests, the gate tests' uses of `sessionEnv` and the env line, the vstest regexes, and the `TestRunTierNoTestNotice` command. The context lacked whether any test pins the wiring of the measuring call, `sessionEnv`, and the console regex shape.
2. `grep` of the `TestRunTierNoTestNotice` command's form and of the darn wiring tests' table builds. The context lacked whether the no-`{args}` vstest path and the darn table build are exercised.

```yaml
lens: code-tests
outcome: reported
```

Task: review the diff's tests under the code-tests lens, listing for each behaviour the diff adds or changes the test row that would fail if it broke.

**Behaviour → pinning test**
- `testreach.ChangedTests`: new and changed tests found and sorted; helpers, methods, `TestMain` and lower-case suffixes excluded; a test moved verbatim or deleted not found; files outside the module, under testdata, `_` or `.` passed over; an unparsable file errors naming it → `TestChangedTests` (internal/testreach/changedtests_test.go:164).
- `declaredFuncsWhere` / `directoryImportPath` refactor kept `ChangedFuncs` the same → existing `TestChangedFuncs` (reach report).
- `gateslot.PackagesFailedOutsideTests` → `TestPackagesFailedOutsideTests` (internal/gateslot/failedtests_test.go:14). The survivor at runlog.go:424 (`start+1` changed) is equivalent: the line it skips is the run-label line, which is never a package line. No finding.
- `gateslot.RedFirstCall` rendering, and the shedrecipe closure passing the row's gate key and site label → **none**. `TestResolveGateSpec_ValidOneEntryPerVocabularyName` only checks that the spec resolves.
- `shedrecipe` `red-first` vocabulary, its required env and the legal-values message → `TestResolveGateSpec_ValidOneEntryPerVocabularyName`, `TestResolveGateSpec_RedFirstRequiresItsEnv`, `TestResolveGateSpec_UnknownNameListsTheLegalValues` (internal/shedrecipe/entries_gate_test.go:61, 381, 371).
- Darn recipe lists `red-first` with 3 attempts on Darn-Write → `TestDarnVerifyBudget` (internal/loomrecipe/darnrecipe_test.go:308).
- `loomcli.entryLabels`: board read, an absent entry read as no label, error wrap → **none**. The `TestWire_*` tests only build the closure.
- `NewRedFirstGate`:
  - no bug label, a "No test seam" record, exit 0, 3, 4 and other codes, a labels error → `TestRedFirstGate_VerdictPerLabelRecordAndExitCode` (internal/loomshed/redfirstgate_test.go:174).
  - timeout and runner-crash branches, record read or parse error, the case-insensitive title match → **none**.
- `gatecli.changedTestsAt` → `TestChangedTestsAt_ComparesTheTestFilesOfEachPackageHoldingAChangedOne`, but the survivor at redfirst.go:287 shows a gap (finding 4).
- `baseOverlay`: production copy, base-lacks → `""`, test, non-Go, outside-module and testdata files left alone → `TestBaseOverlay_ReplacesOnlyTheProductionGoFilesOfTheModule`.
- `writeOverlay` shape → `TestWriteOverlay_HoldsTheReplacementsUnderTheReplaceKey`. The 0o600 survivor is not reasonably killable. No finding.
- `judgeRedFirst`: green, named red, outside-any-test red, no report, another test's failure → `TestJudgeRedFirst_ReadsTheRunsExitCodeAndFailureReport`.
- `runRedFirst` missing-`--gate` refusal → `TestRunRedFirst_RefusesAMissingGateKeyWithAWayForward`.
- `runRedFirst`:
  - non-`go` scoper note with exit 0 → **none**.
  - `TierList` error, no-tier and module refusals → **none**.
- End to end with the real go command: no-test exit 4, red exit 0 naming the test, green exit 3 → `TestGateRedFirst` (tier 1), which also kills the flag survivors at redfirst.go:87/88.
- End to end: a changed test that does not compile on the base counted red, a production file the fix adds deleted by the overlay, the exit-1 unjudged path → **none**.
- `verbDiffBase` refactor (shared refusals) → the existing `ABaseNamingNoCommit` row (internal/gatecli/mutate_integration_test.go:316) through `gate mutate`.

**Findings**

1. MEDIUM · design · `internal/loomcli/wiring.go:591-608`. Nothing tests `entryLabels`, the reader that decides whether the red-first gate runs at all.
   - Untested: the board open, `GetTask` on `seedSlug`, an absent entry read as "no label" (a silent pass), and the error wrapping.
   - The wire tests only check that the seam is non-nil. Reach report: `entryLabels (new)` is reached only by the `TestWire_*` tests.
2. MEDIUM · design · `internal/gatecli/redfirst.go:246-260`. Nothing tests that `runRedFirst` exits 0 with a scoper note when the hub's scoper is not `go`.
   - That path makes the Darn-Write gate pass a bug fix without judging anything.
   - The `--gate` key `TierList` refusal, the no-tier refusal and the module refusal have no test either.
   - Evidence: no gatecli test names `noteScoper` or a red-first refusal (grep); the only refusal row is `redfirst_test.go:833`.
3. MEDIUM · design · `internal/gatecli/redfirst_integration_test.go:546-608`. The real-go test has no row for two behaviours the summary states as design decisions.
   - A changed test that does not build on the base, such as one calling a symbol the fix adds, should count as red. Only the synthetic report row at `redfirst_test.go:811` pins that, so nothing shows that `go test -json` under the overlay produces a "failed outside any test" line for the test's package.
   - A production file the fix adds should be deleted by the overlay's `""` mapping. The integration fix only edits the existing `calc.go`.
   - The exit-1 "judged none" message in `redFirstCall.run` (redfirst.go:359-360) has no row.
4. LOW · design · `internal/gatecli/redfirst_test.go:676`. The mutation survivor at redfirst.go:287 (negating `strings.HasSuffix(rel, "_test.go") && isModulePackage(...)`) passes the test.
   - Every changed test package in the fixture also holds a changed production file (`sub/a/a.go`), so the negated condition still selects `sub/a`.
   - The test needs a row where a package's only changed path is a `_test.go` file.
   - Commit 8399cbd0 says this test "pins which packages' tests red-first compares", which the survivor contradicts.
5. LOW · design · `internal/loomshed/redfirstgate.go:121-126, 83-90, 92`. `TestRedFirstGate_VerdictPerLabelRecordAndExitCode` has no row for:
   - the `TimedOut` and `RunnerCrash` terminal branches of `redFirstFailure`;
   - a decision-record read error or parse error;
   - the case-insensitive "No test seam" match. The fixture title at redfirstgate_test.go:179 uses the exact case, so dropping `ToLower` survives.
6. LOW · design · `internal/shedrecipe/entries_gate.go:335` and `internal/gateslot/redfirst.go:15`. No test asserts what the built gate actually runs: `env.RedFirstCommand(gateKey)` for the row's key, and `RedFirstCall` producing a valid `lym gate red-first --gate <key>` invocation.
   - Passing `""` or the wrong key, or a typo in the rendered verb, passes every test. The gatecli and loomshed tests use their own literal commands, and the fixtures re-spell the string (fixture_test.go:276, envkit.go:423).

Tool calls:
1. grep of the loomcli and gatecli tests for `entryLabels`, the diff-base refusal texts, `red-first` and the scoper or no-tier messages, to find which tests reach those paths; the manifest held definitions, not test call sites.

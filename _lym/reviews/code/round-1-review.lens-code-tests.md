```yaml
lens: code-tests
outcome: reported
```

**Lens: code-tests.** For each behaviour the diff adds or changes, this is the test that pins it, or none. I read test names and some assertions by grep; I did not read every test body.

| Behaviour | Pinning test |
|---|---|
| Records guard: one commit per write, hand-edit mismatch, restore, no-commit unguarded, unknown key survives | `TestRecords` rows in `recordsguard_test.go` |
| Seal check order, `PastRow`, `Unsealed` ignoring a later seal, seal trailer with an extra baseline-digest trailer | `TestSealAndBaseline` in `seal_test.go` |
| `--restore` / `--discard` / entries, drop covered rulings, admitted rows, refusals | `TestAmend` rows in `amend_test.go` |
| Check ordering, baseline fault, records fault, seal fault under lines and command, failed capture, old `partial` leave-out, recorded diff wins over config | `TestCheck` rows in `check_test.go` |
| Drift gate passes or fails per report | `NewDriftGate` assertions at `check_test.go:388` |
| Unkeyed pairing, occurrence, added and removed lines and cases | `TestDiffCaptures` in `diff_test.go` |
| Open and closed ranges, covers-key and covers-line | `expectation_test.go:51` and `:145` |
| Drift-Expect: Done without an agent, seal, listing, digest on first seal, closed and open session marks, Stuck for the parent, exhausted entry respawns, expectation gate | `TestDriftExpect` subtests in `expect_test.go` |
| Drift-Verify: stored rulings reused, judge spawn, does-not-match stored, added-case file, open ranges, resumed session, judge-change Stuck and resume, exhausted judgement, command mode, Recheck on the same engine | `TestDriftVerify` subtests in `verify_test.go` |
| `judgement.yaml` removed after a Done and after a does-not-match Stuck | `os.Stat(f.paths.Judgement)` asserts at `verify_test.go:232` and `:378` |
| Judge snapshot: copies, gone snapshot reads not-found, scratch changes | `TestJudgeSnapshot` in `judgesnapshot_test.go` |
| `Defer` admission at the new rows | `verbs_test.go:62` |
| Old-chair ending only for told roles | `TestStrandRoleStopper_StopsOnlyTheToldRoles` in `driftwiring_test.go:35` |
| Verdict: scope source, deferral, counts, open ranges, amendments, old `partial`, old Drift fields, waiver, hand-edited record | `TestVerdict` in `verdict_test.go` |
| Fabric trailer commits, checkout states, `RecordsPathsChangedSince` | `trailercommit_integration_test.go`, `checkoutstate_integration_test.go:83` |
| Current-capture lock wait (doubling backoff, slot before lock, no record written) | `capture_test.go:192`, `:527`, `:549` |
| Run-level entry report reaches the fixer | `TestWrapRunLevelEntry`, `TestVerifyGateReport_RoundTrip` in `verifygatereport_test.go` |
| Hub-only keys refused in `run_settings` | `configreg/runsettings_test.go` |

I found no behaviour without a pinning test. Two minor points, both from names only:

- **[NIT / consistency]** `internal/driftengine/verbs_test.go:56` — the test is still called `TestDeferWaive`, but the `waive` verb is gone (D12, card 22). Rename it to `TestDefer`; its subtest is named `Defer`.
- **[LOW / design, unverified]** The three reinterpretation flags (D15: `compare` set with no `diff`, `{skip_file}`, `partial`) live in `driftengine/diffmode.go`, and `diffmode_test.go` exists. I did not confirm one row per flag, nor a `hubreconcile` round trip. The handler should confirm a row per flag exists in `diffmode_test.go`.

**Tool calls**
1. A `grep` of test function and subtest names across the drift engine test files, because the manifest held no test files.
2. A `grep` for key behaviours across all test files, to find which test covers each one.
3. A `grep` for reinterpretation, diff, range and judgement tests, to confirm rows for the D15 flags and the diff cases.
4. A `grep` for gate, judgement-removal, verdict and fabric tests, to confirm those assertions exist.

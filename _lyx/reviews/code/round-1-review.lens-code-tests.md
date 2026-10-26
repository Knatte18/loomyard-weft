```yaml
lens: code-tests
outcome: reported
```

**Lens: code-tests. Per behaviour, the test that pins it, or none.**

| Behaviour | Pinning test |
|---|---|
| `rename-leaves-test` flags `Test<base>`, `Test<base>_<case>` (two refs give two findings), attributed to the renaming card | `TestValidate_RenameLeavesTest` (validate_test.go:844), rows 1–2 |
| Method `T.M` matches `TestT_M` | same test, row 3 |
| `TestOldest` is not matched | same test, row 4 |
| A test in another unit is not matched | same test, row 5 |
| A test renamed by its own pair draws no finding | same test, row 6 |
| `csharp` is silent | same test, row 7 |
| `none` is silent | **none** |
| `RemovedTargets` gives one blocking finding per card, for targets and `Uses:` refs | `TestRemovedTargets` (drift_test.go:288), rows 1–2 |
| Quarry's evidence-tier rename candidates are named in the finding | same test, row 2 |
| A reported rename is exempt | same test, row 3, at unit level only |
| A Delete target or a Rename pair's old side on the batch's own cards is exempt | same test, rows 4–5 |
| A removed symbol no later card names draws no finding | same test, row 6 |
| An exact quarry rename is exempt (quarry drops it from `Deleted`) | no direct row; see finding 2 |
| record-batch hands back an OK fork report once, before bind, keeping the batch open with its report and setting the flag | `TestRecordBatch_PostBatchChecks`, subtests at recordbatch_git_test.go:171 and :233 |
| The second record-batch records done with the later-card warning | the same two subtests, second call |
| `later_target_removed` envelope with batch name, a message naming symbol, card and way forward, and the saved flag surviving the CLI state save | `TestRecordBatchCmd_LaterTargetRemovedEnvelope` (verbs_test.go:827) |
| Fabric sync on the hand-back path | none (`FABRIC_SKIP_GIT` is set); not worth a test |
| recover-batch never hands back | new row in `TestRecoverBatch_TerminalRunsTheSamePostBatchChecksAsRecordBatch` (`recoverRemovedLaterTarget`) |
| Hand-back only for an OK report (`HandBackRemovedTargets: report.Status == ReportStatusOK`) | **none** |
| `batchDelta.RemovedTargets` wrapper in `internal/planglyph/index.go` | **none**; see finding 3 |
| `BatchState.LaterTargetsReturned` is read tolerantly | no new test, but it is an additive `omitempty` field; the verbs test round-trips it |

**Findings**

1. **MEDIUM, `design`.** The FAILED-report case is untested.
   - Where: `internal/websterengine/recordbatch.go:570`, which sets `HandBackRemovedTargets: report.Status == ReportStatusOK`.
   - The decision record bounds the refusal as "never on a FAILED or NEEDS_PARENT report".
   - No test records a FAILED report over a removed later target and checks it is not handed back.
   - Dropping the `== ReportStatusOK` condition would break no test.
   - A row in `TestRecordBatch_PostBatchChecks` would pin it.

2. **LOW, `design`.** The exact-rename and report-lists-it exemptions are not pinned at the engine level.
   - Where: `recordbatch_git_test.go:383`.
   - That subtest's mixed rename/delete scenario sets `LaterTargetsReturned = true` first, so it no longer shows that an exact rename draws no hand-back.
   - The subtest at `recordbatch_git_test.go:270` (a reported rename quarry cannot prove) does run unflagged, so it probably covers the reported-rename wiring, which converts `Renames` to `reported`.
   - I did not open its body to confirm.
   - Acceptance criterion "an exact rename draws no refusal" has no direct test. Quarry's exact-rename path may be pinned by other existing exact-rename repair tests, which I did not check.

3. **LOW, `design`.** The real `batchDelta.RemovedTargets` wrapper is not exercised.
   - Where: `internal/planglyph/index.go:112`, which computes `PendingPlan(fullPlan, completed)`.
   - The engine tests use `fakeDelta`, which re-implements the wrapper in `gitfake_test.go:133`.
   - `TestRemovedTargets` calls `RemovedTargets` with a hand-built pending plan, so a wrong `completed` or `own` wiring in the production wrapper breaks no test.

4. **LOW, `design`.** The `none` language is not tested for `rename-leaves-test`.
   - Where: `TestValidate_RenameLeavesTest`.
   - The decision record says the check is silent under `csharp` and `none`, but only `csharp` has a row.
   - `GoToolchain()` is true for an absent language.

**Test economy**
- No duplicate tests found.
- The `recoverbatch_test.go` rewrite extracts the old single subtest into `recoverRemovedLaterTarget` and adds one row (unreported removal). The refactor is needed for the new coverage and is not padding.
- The existing hand-back-sensitive subtests were converted to "hand back once, then warn" rather than copied, which fits `PATTERN-test-speed`.
- The `cli_test.go` change is comment-only.
- The stencil's `later_target_removed` rung has no assertion, which is exempt as static prose.

**Tool calls**
1. `git diff` of the test files, to read the new and changed tests.
2. `git diff` of `recoverbatch_test.go`, because it is a separate large rewrite.
3. `grep` for `RemovedTargets` in tests, to check whether the wrapper and the engine wiring are exercised outside `drift_test.go`.
4. The same command also listed the `recordbatch_git_test.go` subtest names and searched it for FAILED-status rows.

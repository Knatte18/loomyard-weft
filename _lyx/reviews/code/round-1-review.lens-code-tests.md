```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, the test that pins it, or none.

**Per behaviour**
- `ParseReport` requires a non-empty `need` with `NEEDS_PARENT`, refuses a `need` on any other status, and round-trips: `TestParseReport_Rejects` (rows "NEEDS_PARENT without a need", "a need on an OK report") and `TestReport_RoundTrip` (new `NEEDS_PARENT` row). Pinned.
- A fork `NEEDS_PARENT` report distills to a waiting digest and is recorded in flight: `TestRecordBatch_NeedsParentWaits` asserts `Terminal` false, status `waiting`, recoveries unchanged, no `CardSHAs`, `CurrentBatch` kept and the report archived. Pinned.
- `record-batch` on a waiting batch refuses naming the need and `begin-batch 1`: the second half of `TestRecordBatch_NeedsParentWaits`. Pinned.
- `begin-batch` re-begin of a waiting fork batch keeps the start SHA and the recovery count: the new row in `TestBeginBatch_Record`. Pinned.
- A recovery `NEEDS_PARENT` is recorded waiting by `PersistRecoveryTerminal`, `recover-batch` exits 0 with the need, and a resume at the cap is neither attached to, refused as exhausted, nor counted: the extension of `TestRecoverBatchCmd_RunningThenTerminal`.
  - The amendment was rendered by the spawn before the wait. Dropping `waitsOnParent` from the attach, exhaustion or count site would therefore fail the resume assertions. Pinned.
- A dead or stuck envelope carries `recovery_session` and names it: the same test asserts `"recovery_session":"<id>"` and `(session <id>)` on the stuck path. The dead path shares the `default:` branch. Pinned.
- `RenderProgress` waiting line, including `resumeStep` for fork and recovery kinds: the new row in `TestRenderBatchLists`. Pinned.
- The Master digest-field list now includes `need`: updated row in `TestMasterTemplate_QuotesBulletLists`. Pinned.
- `Run` refuses with `ErrBatchDeadEnd` for a dead recovery with no retry and for an exhausted batch, then runs after a batch-start reset: two `deadEndRefusalCase` rows in `TestRun_RefusesBeforeSpawn`. The refusal rows check the error, text and way forward, and the case's reset closure covers the run proceeding. Pinned.
- The Webster adapter maps `ErrBatchDeadEnd` to `Stuck`: the "BatchDeadEnd" row in `TestWebsterProducer_OtherEngineErrors`. Pinned.
- A blocking `Stuck` at a row with no `OnStuck` is budget-exempt and folds with repeats 1: `TestStep_RepeatedBlockingHaltFolds`. Pinned.
- The stencil bodies (`NEEDS_PARENT` mapping, Master message-and-wait): static prose, exempt from coverage. The diff adds no new assertion on it other than the digest-field list row above.
- Tolerant read of an old record with no `need` or `recovery_session`: no new test. It follows from `omitempty` and the existing record tests, so I raise no finding.

**Findings**

1. LOW, consistency: `internal/websterengine/recordbatch_test.go`, around line 1388. `TestRecordBatch_NeedsParentWaits` was inserted between the doc comment of `TestRecordBatch_MalformedReportYAMLErrors` and its function.
   - The two comments now run together above the new test.
   - `TestRecordBatch_MalformedReportYAMLErrors` has no doc comment.
   - Evidence: the diff hunk at `recordbatch_test.go:1387`. Fix: move the new test above that comment block, or move the "TestRecordBatch_MalformedReportYAMLErrors proves…" comment back onto its function.

2. LOW, scope: `internal/websterengine/runlevel_test.go`, `TestRun_RefusesBeforeSpawn` rows. The two bounds the decision says keep the refusal narrow have no negative case.
   - Dead with `RecoveryRetry` true.
   - Terminal at the cap with an unrendered amendment.
   - A `done` batch.
   - The decision record says it "fires only on the states for which Master's own ladder already ends the run stuck". A change that widened `deadEndBatchError` (for example dropping `!rendersNewAmendment` or the `Status == done` skip) would pass every test.
   - Fix: add a non-refusing table of the three cases, each reaching Master. This can be rows in one test, not one test each.

No duplicated coverage or per-helper tests found.

**Tool calls**
1. `git diff fe9fbafc3..HEAD -- '*_test.go'`: the manifest held only non-test diffs.

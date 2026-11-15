```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, the test that pins it, or none.

**Per-behaviour list**
- Body edit on a discussed entry with a stored record returns a warning. Pinned by `TestUpsertTaskReportingRecordWarning/BodyEditOfADiscussedTaskWarns`. It checks that the warning names the slug, `--record-file` and `lym loom decision add filed`, and that the body is written.
- An entry with no stamp warns on a body change. Pinned by `TestUpsertTaskReportingRecordWarning/BodyEditOfAnUnstampedDiscussedTaskWarns`.
- A title-only edit is silent (the `before.Body == after.Body` guard). Pinned by `TestUpsertTaskReportingRecordWarning/TitleEditOfADiscussedTaskIsSilent`.
- Restoring the stored body is silent (the stamp-equals-digest guard). Pinned by `TestUpsertTaskReportingRecordWarning/BodyRestoredToTheStoredBodyIsSilent`.
- An undiscussed entry is silent (the `!Discussed` guard). Pinned by `TestUpsertTaskReportingRecordWarning/BodyEditOfAnUndiscussedTaskIsSilent`.
- The warning never refuses or alters the write. Pinned by the `err == nil` and body-written assertions in `TestUpsertTaskReportingRecordWarning`.
- The CLI `upsert` envelope carries a `warning` key naming `--record-file`. Pinned by `stepUpsertAndPromoteWarnOnChangedSketchBody`, via the `TestCLI` step.
- The hand-built envelope keeps `task` and `notice`. Pinned by the existing `TestCLI` steps and the sibling-notice tests, which I did not re-read.
- `TestWriteSiblings` now goes through `UpsertTaskReporting`, so the siblings result is still pinned.
- A done entry is silent (the `isDone(after)` guard). **None.** No row builds a done discussed entry with a stored record and edits its body.
- A discussed entry whose record file is missing is silent (the `recordStored` guard). **None.** The only silent row with no record is the undiscussed one, which the `!Discussed` guard short-circuits first, so removing `recordStored` would not fail any test.
- An upsert with `--record-file` carries no `warning` key (`UpsertTaskWithRecord` leaves `RecordWarning` empty). **None.** My grep of `stepUpsertRecordFile` found no `warning` assertion.
- A CLI `upsert` with no record-relevant change omits the `warning` key. **None** at the CLI level. The `steady` entry is only promoted, never upserted with a body edit. The engine rows cover the empty-warning case.
- A new entry (the `existed` guard) is silent. No distinguishing test, and none is needed: with an empty `before`, the `!Discussed` guard already gives silence, so the guard can't be observed.

**Findings**

1. **MEDIUM, design.** Location: `internal/boardengine/sketch_test.go` (`TestUpsertTaskReportingRecordWarning` table), guard at `internal/boardengine/decision.go:91-92`. Summary: the `isDone` guard and the `recordStored` guard of `bodyEditRecordWarning` have no row that would fail if either were dropped. The board entry's expected cases include a landed entry and a missing record file. Fix: add two rows. One edits the body of a done entry that has a stored record. The other removes the record file of a discussed entry and then edits its body.

2. **LOW, design.** Location: `internal/boardcli/verbs_test.go` (`stepUpsertRecordFile`). Summary: nothing asserts that an upsert with `--record-file` returns no `warning`. That path is deliberately left without one, because it replaces the record itself. Fix: assert in `stepUpsertRecordFile` that `result["warning"]` is absent.

**Tool calls (2)**
1. Ran grep on `stepUpsertRecordFile` for a `warning` assertion, because that function body was not in my context. Result: none.
2. Ran the same command's second grep for done or warn rows in the boardengine tests. Result: none.

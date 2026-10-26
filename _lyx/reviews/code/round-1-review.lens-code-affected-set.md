```yaml
lens: code-affected-set
outcome: reported
```

**Affected-set list (per change):**

- **`planindex.Delta.RemovedTargets` (new interface method).**
  - Implementers: `planglyph.batchDelta` at `internal/planglyph/index.go:113` and the test fake `fakeDelta` in `internal/websterengine/gitfake_test.go`, which the diff extends by 4 lines. Covered.
  - Callers: `postBatchChecks` in `internal/websterengine/recordbatch.go` only. Covered.
  - No other `Delta` implementer turned up in a grep for the sibling method `ScopeGuard(`. Covered.
- **`BatchState.LaterTargetsReturned` (new field).**
  - Writers: set only in `postBatchChecks` (`recordbatch.go:997`).
  - Reset: `begin-batch` builds a fresh `BatchState` (`beginbatch.go:645`), so the flag clears per bracket, matching the "once per bracket" wording in the docs and error text.
  - Recovery: `recoverbatch.go:388` builds a new recovery record. `recover-batch` never passes `HandBackRemovedTargets`, so it never refuses. Covered.
  - Read tolerantly: it is an `omitempty` JSON bool. Covered.
- **`postBatchInputs.HandBackRemovedTargets` (new field).** Its one construction site is `RecordBatch` (`recordbatch.go:~558`), and the recover path does not set it. Covered.
- **`ErrLaterTargetRemoved` / `LaterTargetRemovedError` (new refusal).**
  - Caller: the `RecordBatch` error branch passes any error other than `ErrCardNotDone` straight through (`return nil, err`), so the batch stays open with its report. Covered.
  - Consumer: the `webstercli` record-batch branch, which handles it before `ErrReportArchived`. Covered.
  - Docs: `websterengine/doc.go:318`. Covered.
- **CLI envelope key `later_target_removed`.**
  - Consumers: the Master stencil rung (line 170 and the done-check paragraph at 248), the record-batch help text, and the envelope test `verbs_test.go:827`.
  - No other doc, fixture or consumer lists envelope keys. A repo-wide grep for `report_archived`, the sibling key, finds only the stencil, the CLI and its test. Covered.
- **`rename-leaves-test` plan check.**
  - Spec: row 53 in `contracts/specs/loom-plan-spec.md`, plus its position note and the csharp skip list. Covered.
  - Package docs: `planparser/doc.go` (the `GoToolchain` list and the new paragraph) and the `validate.go` header list. Covered.
  - Plan writer stencil: `loom-template-plan.md`. Covered.
  - Check order: the call sits right after `checkRenameMechanicMissing`, as the spec says. Covered.
- **`RemovedTargets` finding `later-target-removed`.** Documented in `planglyph/doc.go`. Covered.
- **Stencil wording.**
  - Master stencil: the new rung matches the decision record's flow (SendMessage, then record-batch again, with the unreachable-fork case). The done-check paragraph is amended consistently, and no existing rule is dropped.
  - Implementer body: a new "A batch handed back" section, a "unless your parent hands the batch back" qualifier on the write-once rule, and a rename-test line. Matches the record, and no existing rule is weakened. Covered.

**Findings:** none. I found no member of the affected set that the diff leaves wrong.

**Tool calls (3):**
1. `grep` for `ScopeGuard(` func definitions, `BatchState{` constructors, and `git diff -- contracts`. My context lacked the other `Delta` implementers, the places a `BatchState` is built (for the flag reset), and the stencil and spec diffs.
2. `grep -rln report_archived`. My context lacked the other consumers of the sibling envelope key.
3. `grep -rn later_target_removed`. My context lacked every place the new key appears.

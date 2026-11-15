# Warn when an upsert edits the body of a discussed entry

A run of a discussed entry, such as a task a raddle filing left with `discussed: true`, writes its decision record from the stored record and never reads the board body.
A body edit made after the store therefore reached no run, and `lym board upsert` saved it with no word (issue #121).

`lym board upsert` now adds a `warning` to its success envelope when the write changes the body of an open entry whose decision record file exists, and the entry's `record_body_sha256` stamp is not the new body's digest.
The write still lands.
The warning names the two ways to carry the edit into the record:
before the run starts, fold it into the record `lym board get --record` prints and store it with `lym board upsert --record-file`;
once the run has a task worktree, record it with `lym loom decision add <slug>`.
An entry stored before the stamp existed carries no stamp and warns on any body change.
A write that leaves the body alone, or restores the body the record was stored against, stays silent.

## Design decisions

- **Warn, not refuse.**
  The issue allows either.
  A refusal would block an operator who means to edit the body first and the record after, and the promote of a Sketch already warns on the same stamp mismatch, so the upsert follows it.
- **Engine API.**
  `Board.UpsertTaskSiblings` becomes `Board.UpsertTaskReporting`, returning an `UpsertResult` that carries the task, the siblings and the record warning, mirroring `PromoteResult`.
  The judgment lives in `bodyEditRecordWarning` in `internal/boardengine/decision.go`, beside the stamp helpers.
- **Scope.**
  Only the single `upsert` warns; `upsert-batch` and `merge` are unchanged.
  An entry a run holds already refuses a body change through the run lock, so the warning covers the window between the store and the run's start.
- **Not taken:** making Plan-Write and the plan review read the body beside the record.
  That would change the recipe contract of every discussed run; the warning closes the silent path at the point of the edit.

## Tests

- `TestUpsertTaskReportingRecordWarning` in `internal/boardengine/sketch_test.go` covers a body edit on a stamped and an unstamped discussed task (warns), a title-only edit, a body restored to the stored one and an undiscussed task (silent).
- The boardcli step `stepUpsertAndPromoteWarnOnChangedSketchBody` (renamed from `stepPromoteWarnsOnChangedSketchBody`) now also asserts the upsert envelope's `warning`.

# Warn when an upsert edits the body of an entry with a stored record

A run of a discussed entry plans from its stored decision record and never reads the board body.
A body edit made through `lym board upsert` before the run starts therefore never reached the run, and the upsert gave no sign of it.

`lym board upsert` without `--record-file` now adds a `warning` key to its envelope when the write changes the body of an open, discussed entry whose record file exists.
The write still lands; the upsert never refuses on it.
The warning names the slug and two ways to carry the edit into the record:

- before the run starts, fold the edit into the record `lym board get --record` prints and store it with `lym board upsert --record-file`;
- once the run has a task worktree, record it with `lym loom decision add`.

The warning is silent for these writes:

- a write that leaves the body alone;
- a write that restores the body the record was stored against, judged by the entry's `record_body_sha256` stamp;
- an entry that is done, undiscussed, or has no record file;
- `upsert --record-file`, which replaces the record itself.

An entry with no stamp warns on any body change.

Closes #121

## Design decisions

- The warning option was taken over reading the body beside the record in Plan-Write and the plan review.
  The stored record stays the single source a run reads.
- `boardengine.UpsertTaskSiblings` is replaced by `UpsertTaskReporting`, which returns an `UpsertResult` holding the task, the siblings and the `RecordWarning`.
  `Board.UpsertTask` wraps it unchanged.
- The check compares the entry before and after the write inside the write lock, in `bodyEditRecordWarning` (`internal/boardengine/decision.go`).
- The upsert envelope in `internal/boardcli/cli.go` is now built directly so it can carry `warning` beside `task` and `notice`.
  The `promote` warning for a changed Sketch body is unchanged.
- The board skill and the `boardcli` and `boardengine` package docs describe the new warning.

## To check by hand

- `body-edit-warning-guards-untested` (`internal/boardengine/siblings_test.go`): confirm `TestUpsertTaskReportingRecordWarning` pins the done and missing-record-file silent cases and the unstamped case.
- Run `lym board upsert '{"slug":"<discussed slug>","body":"…"}'` against a discussed entry and confirm the envelope still has `ok` and `task`, now with `warning`.

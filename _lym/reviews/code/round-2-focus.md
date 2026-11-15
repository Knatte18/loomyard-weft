---
round: 2
focus:
  - "In `internal/boardengine/sketch_test.go`, `TestUpsertTaskReportingRecordWarning` exercises `bodyEditRecordWarning` in `internal/boardengine/decision.go`. Settle whether dropping the `isDone(after)` guard, or the `recordStored` guard, would fail some table row, for a done discussed entry with a stored record and for a discussed entry whose decision record file is missing."
  - "In `internal/boardcli/verbs_test.go`, `stepUpsertRecordFile` runs `upsert --record-file`, which replaces and stamps the record itself. Settle whether any assertion fails if that path starts emitting a `warning` key."
  - "Settle whether `internal/boardengine/doc.go` and `plugins/ly/skills/board/SKILL.md` match the diff: no reference to the removed `UpsertTaskSiblings` is left, and the skill tells an agent that an upsert of a body edit on an entry with a stored record can add a `warning` key."
---

The focus lists where the previous round's findings pointed, as questions to settle against the code as it stands.

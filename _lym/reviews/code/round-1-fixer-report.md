# Fixer-report, round 1

## Changes

- F1 (fixed as proposed): `TestUpsertTaskReportingRecordWarning` in `internal/boardengine/sketch_test.go` gained table fields `markDone` and `removeRecordFile`.
  New silent rows `BodyEditOfADoneDiscussedTaskIsSilent` and `BodyEditOfADiscussedTaskWithNoRecordFileIsSilent` pin the `isDone` and `recordStored` guards.
- F2 (fixed as proposed): `internal/boardengine/doc.go:700` names `UpsertTaskReporting` instead of `UpsertTaskSiblings`.
- F3 (fixed as proposed): `plugins/ly/skills/board/SKILL.md` documents the `warning` key of an upsert that edits the body of an open entry with a stored record.
- F4 (fixed as proposed): `stepUpsertRecordFile` in `internal/boardcli/verbs_test.go` asserts the `--record-file` upsert result has no `warning` key.
- Self-check change: the doc comment of `TestUpsertTaskReportingRecordWarning` lists the two new silent cases.

`lym gate test ./internal/boardengine ./internal/boardcli` passed (935 passed, 0 failed) before the self-check; the later change is a comment only.

## Self-check

- fix-consistency, LOW: the test doc comment omits the two new silent rows. Fixed as proposed.
- fix-consistency, LOW: the skill text does not say a done entry's body edit is silent. Disputed, premise false: the new text says "open entry", which already excludes done entries.
- fix-code-correctness: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

- Self-check finding on `SKILL.md` (fix-consistency, LOW), ground `premise false`: the added sentence reads "open entry", so it already states that a done entry is outside the warning.

## Deferred

None.

## Rulings

None.

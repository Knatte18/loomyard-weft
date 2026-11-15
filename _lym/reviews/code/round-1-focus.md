---
round: 1
focus:
  - "internal/boardcli/cli.go upsert: the hand-built envelope replaces outputSuccessWithTask. Does the output keep the same task/notice shape and any other key that helper wrote, so existing callers and output tests still hold?"
  - "internal/boardengine/decision.go bodyEditRecordWarning: check each guard (body unchanged, not discussed, done, stamp equals new body digest, record file missing) against the board entry's expected cases. Is an unstamped record warned on, and is a body restored to the stored one silent?"
  - "internal/boardengine/siblings.go UpsertTaskReporting: store.GetTask(fields[\"slug\"]) takes an untyped value and the warning is computed inside the critical section. Does a missing, non-string or newly created slug stay correct and never fail the write? Does the warning ever refuse or alter the write?"
  - "Scope against the board entry: the entry accepts a warning at upsert that names lym loom decision add or another way to amend the record before the run starts. Does the warning text name a way that works before the worktree exists, and is the --record-file path it names real?"
  - "The --record-file upsert path (UpsertTaskWithRecord): is it correctly left without a warning, since it replaces the record itself? Does the removed UpsertTaskSiblings leave any caller or doc reference behind?"
  - "Docs landing: internal/boardcli/doc.go and internal/boardengine/doc.go changes against the repository's constraints document. Do they match the behavior, and are the new doc comments compliant in content?"
  - "Test economy: TestUpsertTaskReportingRecordWarning covers the engine. Check the verbs_test.go, cli_test.go and siblings_test.go edits for overlap with it, and for any new behavior left uncovered, such as the CLI envelope's warning key."
---

Seed focus for round 1, set from the decision record's board entry and the committed diff.

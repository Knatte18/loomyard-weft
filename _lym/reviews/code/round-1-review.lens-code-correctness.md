```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens. For each changed function or branch, the case I ran and whether it was correct.

1. `UpsertTaskReporting` (`internal/boardengine/siblings.go:148`), new slug. `GetTask` returns `existed=false`, so no warning is computed and the write lands. Correct.
2. `UpsertTaskReporting`, nil or missing slug. `GetTask(nil)` finds no entry, and `store.UpsertTask` then fails slug validation. The error comes back as `UpsertResult{}` and the warning is never read. Correct.
3. `UpsertTaskReporting`, `store.UpsertTask` fails. It returns `nil, err` before any warning is set. The `warning` variable stays empty and is discarded with the error. No state is left inconsistent. Correct.
4. `UpsertTaskReporting`, save fails after the mutation. `writeReportingSiblings` returns the error, so the result and warning are dropped. Correct.
5. `UpsertTaskReporting`, `before` snapshot. `before` is a `Task` value, and `Body` is a string, so the later in-place update of `s.tasks[i]` cannot alter it. Correct.
6. `bodyEditRecordWarning`, body unchanged (title-only edit). `before.Body == after.Body`, so it returns empty. Correct.
7. `bodyEditRecordWarning`, undiscussed entry. `!after.Discussed` returns empty. Correct.
8. `bodyEditRecordWarning`, done entry. `isDone(after)` returns empty. Correct.
9. `bodyEditRecordWarning`, body restored to the stored one. The stamp equals `bodySHA256(after.Body)`, so it returns empty. Correct.
10. `bodyEditRecordWarning`, unstamped record with an edited body. The stamp is `""`, which is not the digest, so it warns. This is the documented behaviour. Correct.
11. `bodyEditRecordWarning`, flagged entry whose record file is missing. `recordStored` is false, so it returns empty. Correct.
12. `bodyEditRecordWarning`, `Sprintf` format. Four `%s` or `%q` verbs take four arguments, in the right order. The slug shape is validated by the store, so it cannot break the quoting inside the single-quoted JSON. Correct.
13. `UpsertTask` (`internal/boardengine/board.go:205`). It now discards the warning and returns `result.Task, err`, with the same observable result as before. Correct.
14. CLI upsert, `--record-file` path (`internal/boardcli/cli.go:660`). `result.Task` and `result.Siblings` are filled, `RecordWarning` stays empty, and no warning key is emitted. This is right, because that path replaces the record and stamps it. On error it returns before using `result`. Correct.
15. CLI upsert envelope, no notices and no warning. The envelope is `{"task":…}` only. That matches `outputSuccessWithTask`, which added `notice` only when non-empty. Correct.
16. CLI upsert envelope, notices and warning together. `notice` is the joined notices, and `warning` is a separate key, the same shape as promote's. Correct.

I found no correctness defects.

Out of my lens: `internal/boardengine/doc.go:700` still names `UpsertTaskSiblings`, which the diff removed. That belongs to the affected-set lens.

Tool calls: none. The manifest already held every definition I needed.

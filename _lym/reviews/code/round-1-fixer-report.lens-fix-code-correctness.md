```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the F1 fix (the test fixture in `internal/boardengine/handles_test.go`) for correctness through the fix-code-correctness lens.

No findings.

Walk-through with concrete input:
- The fixture adds entry 9, `{"slug":"nohandle","kind":"note","labels":["removed-type"]}`, and entry 10, a task with `"project":"nohandle"` and `"handle":"T11"`.
- `ReadTasks` leaves the note's handle empty, because the note has no configured type. The passing gate confirms this: a `P1`-style handle assigned to it would have put `under-nohandle` in the result, as happened with the earlier project-kind fixture.
- In `WorkspaceFolderNames`, `handles["nohandle"]` is `""`, and entry 10 has `Project != ""`, so only the `handle != ""` check excludes it. The note itself has no project and is left out.
- `T11` collides with no other task handle (`T4`, `T5`, `T7`, `T10`, `T12`), so it is kept.
- The `AnUnreadableBoardNamesNoEntry` and `NoBoardNamesNoEntry` rows don't read the fixture, so they are unaffected.

Test strength:
- If the `handle != ""` check were dropped, entry 10 would be named `": under-nohandle"`. `maps.Equal` against the unchanged `want` map would then fail, so the test fails with the defect in place.
- The gate for `./internal/boardengine` is green after the fix, with 0 failures. The earlier project-kind fixture was the failing attempt, and the fixer replaced it.

Callers: only the test file changed, so no signature or behavior is at risk.

Tool calls made: none, because my context already held every file and result this lens needed.

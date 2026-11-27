```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, find the test row that would fail if it broke.

**Per behaviour, the test that pins it**

1. `CheckParent` allows a write to the first, earlier path in `notePaths`: pinned by `TestCheckParent` "write to the note an earlier Master's prompt named is allowed" (`audit_test.go:411-417`). It writes `notePath`, the first of two entries, so an implementation that checks only the last entry fails it.
2. `CheckParent` allows a write to the later entries in `notePaths`: pinned by `TestRecordBatch_AuditOutcomes` "a Master write to any saved friction note path…" (`recordbatch_test.go:585-592`), which writes both listed paths. It also pins that record-batch passes `State.MasterNotePaths`.
3. Empty `notePaths` allow no note: pinned by `TestCheckParent` "with no note named, a write to Master's friction note is a hard error" (`audit_test.go:418-423`).
4. Told and link-resolved spellings still match through the list: pinned by the existing link-resolved subtest, now passing `[]string{told}` (`audit_test.go:495`).
5. `Run` appends the spawned prompt's path and keeps the earlier one, in spawn order: pinned by `TestRun_DoneOutcome` "a resumed Master's write to the note an earlier Master's prompt named leaves the run done" (`runlevel_test.go:2039-2068`), which asserts `[earlier, latest]`. It also pins that the run-exit audit passes the list, through its outcome check and its check for no parent-write warning.
6. A first spawn records exactly the prompt's note: pinned by `TestRun_MasterSpawn`'s persisted-identities row (`runlevel_test.go:1675`).
7. `Run` skips a path already in `MasterNotePaths`, the `!slices.Contains` guard at `runlevel.go:810`: **none**. That case arises when an earlier Master never wrote its note and `friction.NotePath` returns the same path again (`friction.go:146-148`). The resumed row creates the earlier note on disk (`runlevel_test.go:2047`), so its resumed prompt names `-2`. No row reaches the duplicate case.
8. `Run` appends nothing when friction notes are off and keeps the earlier paths on such a resume, the `masterNotePath != ""` guard at `runlevel.go:810`: **none**. Only `runlevel_test.go:1675` and `:2050`/`:2064` assert `MasterNotePaths`, and both run with a friction dir. Dropping the guard would store `""` in the list.

**Findings**

- LOW, design, `internal/websterengine/runlevel_test.go:2039`: the duplicate-skip the diff adds at `runlevel.go:810` has no covering row.
  - Fix: add a `TestRun_DoneOutcome` row whose seeded state already lists `webster-master.md`, with no file at that path, and assert `MasterNotePaths` stays a single entry.
  - Evidence: `runlevel.go:810`, `friction.go:146-148`, `runlevel_test.go:2047`.
- LOW, design, `internal/websterengine/runlevel_test.go:2039`: the friction-off branch of the append at `runlevel.go:810` has no covering row.
  - It covers both halves: no `""` entry is stored, and the earlier paths survive a resume with friction notes off.
  - Fix: add a row, or an assertion in an existing row with an empty `FrictionDir`, that checks `MasterNotePaths` after the run.
  - Evidence: `runlevel.go:752`, `:810`; `friction.go:139-140`; the grep of `MasterNotePaths` assertions in `runlevel_test.go`.

None of the new rows duplicates another test's coverage. `TestCheckParent`'s new rows test the pure function, the record-batch row tests that caller, and the resumed `TestRun_DoneOutcome` row tests the run-exit caller and the state lifecycle. Each of the three would fail if the single-path behaviour came back.

Tool calls: none, since the inherited context covered every test and code span this lens needed.

```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens, listing for each behaviour the diff adds or changes the test that pins it.

**Per behaviour, the test that pins it**

1. Archived judgement on the records checkout is exempt, with and without the `-<n>` collision suffix. Pinned by `TestJudgeSnapshot/ArchivedJudgementIsNoChange` (`judgesnapshot_test.go`). It writes both names as records-side untracked files and expects no changes.
2. Siblings that are not archived judgements stay named. The rows are `judgement-notes.yaml` (no stamp), the wrong extension, and a different directory. Pinned by `TestJudgeSnapshot/OtherFileBesideTheJudgementIsNamed`.
3. Archived judgement on the code checkout is exempt. This is the changed predicate in `Changes` at `judgesnapshot.go:207`. Pinned by none.
4. The archived name must belong to the judgement's own base name, which is the `archivedFrom == ...` comparison at `judgesnapshot.go:245`. Pinned by none.
5. `loomRecordsDirs` holds the anchored usage directory, for a root anchor and a subdirectory anchor. Pinned by `TestLoomRecordsDirs_HoldTheUsageDirectoryUnderTheAnchor`. The `export_test.go` re-export is needed by this test, since the function is unexported and the test is external.
6. `isLoomRecordsPath` treating the usage entry as a directory prefix, with no match on sibling names that merely share the string prefix. The diff does not change this function, and I did not check whether another test covers it. The diff adds no row for it.

**Findings**

- **LOW, class design.** `internal/driftengine/judgesnapshot.go:207`: the archived-judgement exemption on the code checkout has no test. All new rows use `f.records`. Reverting the code side to `rel == judgementRel` fails no test. Fix: add a row to `TestJudgeSnapshot` that puts an archived judgement name in `f.code.Untracked` and expects no change.
- **LOW, class design.** `internal/driftengine/judgesnapshot.go:245`: the check that the archived name's base equals the judgement's base has no failing row. The only negative row with a stamp has the wrong extension or directory, and `judgement-notes` fails the stamp pattern first. Dropping the base comparison, so any `<x>-<stamp>.yaml` in `_lym/drift/` is exempt, keeps every test green. Fix: add an untracked row such as `_lym/drift/other-20261106T083132Z.yaml` to the expectations of `OtherFileBesideTheJudgementIsNamed`, expecting it to be named as added.

I found no redundant tests. I did not flag the wiring test and the snapshot test being separate, since each half is pinned at its own public surface.

**Tool calls:** none. Everything I needed was already in context.

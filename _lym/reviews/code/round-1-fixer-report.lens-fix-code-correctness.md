```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the correctness of the fix's test-only changes (`verbs_test.go` comment move, `projectrelease_test.go` rows for F2 and F3).

**Findings**

1. **LOW** — `TestSetStatusReleasingProject_OtherWritesReleaseNoMember`, row `NullStatusOnTask` (`projectrelease_test.go`). The row does not pin the `current.Kind != KindProject` half of the guard at `projectrelease.go:36`.
   - Evidence: remove that half and run the row. `d1` is a draft task, so `draftMembers(tasks, "d1")` is empty, `IsEmptyStatus(draft)` is false, and the code falls to `s.SetStatus(selector, nil)`. That gives the same `d1 = <none>`, `p = ""`, `d2 = draft` and an empty `released` as the guarded path.
   - The `status != nil` half is pinned by `NonNullStatusOnProject`. With the guard gone, `p` has an empty status, so it would be released to `<none>` and `d1` and `d2` would be released too, failing the row.
   - A task whose status is a run status or `done` would diverge. Without the Kind check it would hit `projectReleaseRefusal` (`projectrelease.go:40-41`), so a row on such a task would fail with the defect in place.
   - Why only LOW: the review asked for "a null status on a task", and the row does that.

**Checked, no findings**
- `RunStatusProjectNamesEveryMember` asserts the exact `"; "`-joined string in store order `d1`, `d2`. A different separator or order fails it.
- `DoneProjectWithoutDraftMember` asserts `has no draft member task to release`. Removing the `len(members) == 0` branch in `projectReleaseRefusal` makes the message differ, so the row fails.
- `NonNullStatusOnProject` on a project with an empty status, as analysed in finding 1, fails if the `status != nil` half of the guard is removed.
- The Refusals loop now checks each slug in `tt.members` is still draft. The old `d1`-only check is covered by the `MemberFailingValidation` and `DoneProject` rows, which keep `members: ["d1"]`. Board ids are unchanged for those rows.
- The `write` signature change to `([]string, error)` has every row returning a matching value, and the `SetStatus` and `UpsertTask` rows return `nil` for `released`. The `fmt` import is used.
- The `verbs_test.go` comment move leaves `stepProjectRelease` with its own two-line comment and `stepProjects` with its full block directly above its `func`. No code changed.

**Further tool calls:** none made; the context already held every changed file and `projectrelease.go`.
